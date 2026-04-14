---
name: asset-loading
description: Use when loading dynamic resources, prefabs, environments, or audio assets at runtime in Unity. Also activates proactively when the AI detects Resources.Load, Resources.LoadAsync, or direct asset path strings anywhere in the codebase — these must always be migrated to Addressables.
---

# Skill: Addressables & Asset Pipeline

---

## Core Rules

| Rule | Requirement |
|---|---|
| `Resources.Load()` | **Forbidden** unless a legacy system explicitly requires it (document why) |
| Asset instantiation | Must use `Addressables.InstantiateAsync()` |
| Asset loading (non-prefab) | Must use `Addressables.LoadAssetAsync<T>()` |
| Error handling | Every `AsyncOperationHandle` must check `Status == AsyncOperationStatus.Succeeded` |
| Memory release | Every loaded asset MUST have a paired release call — no orphaned handles |

---

## Execution Sequence

### Step 1 — Addressable Setup Verification
Before coding any loading logic, confirm:
- The asset has an **Addressable Address** assigned in the Inspector
- It belongs to the correct **Addressable Group** (see Group Strategy below)
- **Labels** are assigned if the asset needs batch loading

### Step 2 — Safe Asset Loading Pattern

```csharp
public sealed class EnemySpawner : MonoBehaviour
{
    [SerializeField] private AssetReferenceGameObject _enemyRef;
    private AsyncOperationHandle<GameObject> _handle;

    public async UniTask<GameObject> SpawnAsync(Vector3 position, CancellationToken ct)
    {
        _handle = Addressables.InstantiateAsync(_enemyRef, position, Quaternion.identity);

        await _handle.Task.AttachExternalCancellation(ct); // UniTask integration

        if (_handle.Status != AsyncOperationStatus.Succeeded)
        {
            Debug.LogError($"[EnemySpawner] Failed to load asset: {_handle.OperationException}");
            return null;
        }

        return _handle.Result;
    }

    private void OnDestroy()
    {
        if (_handle.IsValid())
            Addressables.ReleaseInstance(_handle);
    }
}
```

### Step 3 — Batch Loading via Labels
When loading a category of assets (all enemies, all UI icons):

```csharp
private async UniTask LoadEnemyAssetsAsync(CancellationToken ct)
{
    var handle = Addressables.LoadAssetsAsync<GameObject>(
        "EnemyPrefabs",
        asset => _enemyCache.Add(asset) // Callback per-asset as they load
    );

    await handle.Task.AttachExternalCancellation(ct);

    if (handle.Status != AsyncOperationStatus.Succeeded)
    {
        Debug.LogError($"[AssetLoader] Batch load failed: {handle.OperationException}");
        Addressables.Release(handle);
        return;
    }

    _batchHandles.Add(handle); // Track for bulk release later
}
```

### Step 4 — Loading Screen Integration
For scene transitions requiring a loading screen:

```csharp
public async UniTask LoadSceneAsync(string addressableSceneKey, IProgress<float> progress, CancellationToken ct)
{
    var handle = Addressables.LoadSceneAsync(addressableSceneKey, LoadSceneMode.Single);

    while (!handle.IsDone)
    {
        progress?.Report(handle.PercentComplete);
        await UniTask.Yield(ct);
    }

    if (handle.Status != AsyncOperationStatus.Succeeded)
        throw new Exception($"Scene load failed: {addressableSceneKey}");
}
```

### Step 5 — Memory Release Enforcement
**A loading script without a paired release path is an architectural violation.** Choose the correct release mechanism:

| Scenario | Release Method |
|---|---|
| Instantiated prefab | `Addressables.ReleaseInstance(gameObject)` |
| Loaded asset (non-prefab) | `Addressables.Release(handle)` |
| Batch-loaded assets | `Addressables.Release(batchHandle)` |
| Scene | `Addressables.UnloadSceneAsync(handle)` |

Always trigger release in `OnDestroy()` or a custom `IDisposable.Dispose()` — never rely on GC.

---

## Addressable Group Strategy (Mobile)

| Group Name | Content | Update Behavior |
|---|---|---|
| `Built-In` | Core UI, essential prefabs | Packed with build |
| `Remote-Static` | Level art, cutscenes | Remote CDN, content-hash cached |
| `Remote-Dynamic` | Seasonal/event content | Remote CDN, always check |
| `Preload` | Critical gameplay assets | Downloaded on first launch |

---

## Mobile Memory Budget Guidelines
- A single mobile scene should load no more than **150–200 MB** of GPU memory
- Unload assets aggressively when transitioning scenes — don't hoard handles
- Use `Addressables.CheckForCatalogUpdates()` on app resume for live-ops content

---

## Cross-Skill References
- Scene transition patterns → `skills/scene-management`
- Error handling and async patterns → `rules/07-async-coroutines.md`
- Memory profiling of loaded assets → `skills/unity-profiler-mind`
- Wrapping loader in a service → `skills/create-feature` (Step 3)
