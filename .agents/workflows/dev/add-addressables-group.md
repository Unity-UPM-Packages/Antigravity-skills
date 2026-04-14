---
description: Create a new Addressables group with Decision Matrix, generic loader with handle leak prevention, interface contract, and pre-warm pattern.
---

# Workflow: Add Addressables Group

Add a new Addressables Group to manage a specific category of assets (e.g., Enemy Prefabs, Level Environments, UI Bundles, Audio Clips). Ensures runtime loading is correct, labeled, backed by a loader script, and free of any `Resources.Load` calls.

---

## Step 1 — Define the Group (Decision Matrix)

Answer all 4 questions, then apply the corresponding configuration:

```
1. What type of assets does this group contain?
   (Prefabs / Textures / Audio / ScriptableObjects / Scenes)
   → Determines the generic type T in the loader script.

2. When are they loaded?
   at boot              → Apply pre-warm pattern (Step 9A)
   on level start       → Load during loading screen, before gameplay begins
   on-demand in-game    → Load on-demand with CancellationToken

3. Should individual assets be released after use?
   yes                  → Track handles per-address (default loader pattern)
   no / entire group    → Load all, dispose entire group at scope end

4. Approximate asset count?
   < 10                 → Bundle Mode: Pack Together
   10–100               → Bundle Mode: Pack Together (same usage pattern)
   > 100                → Bundle Mode: Pack Separately (allows granular release)
```

---

## Step 2 — Create the Addressables Group

Open **Window → Asset Management → Addressables → Groups**:

1. Click `Create → Group → Packed Assets`
2. Name the group: `[Category]_[Type]` (e.g., `Enemies_Prefabs`, `UI_Backgrounds`, `Audio_Music`)
3. Configure Group Settings:
   - `Build Path`: `[UnityEngine.AddressableAssets.Addressables.BuildPath]`
   - `Load Path`: `[UnityEngine.AddressableAssets.Addressables.RuntimeLoadPath]`
   - `Bundle Mode`: from Decision Matrix Step 1.4 above

---

## Step 3 — Assign Assets to the Group

For each asset to manage:
1. Select the asset in the Project window
2. In Inspector: tick the **Addressable** checkbox
3. Open the Addressables Groups window → drag the asset into the correct Group
4. Set a concise **Address** in `category/name` format (e.g., `enemies/slime`, `ui/shop-background`)

---

## Step 4 — Create Labels

In the Addressables Groups window → **Labels** dropdown:
1. Add a new label: `[category]` (e.g., `enemies`, `ui-backgrounds`, `music`)
2. Assign the label to all assets in the group

Labels allow batch-loading an entire category with a single call.

---

## Step 5 — Define the Interface (Interface First)

Create `Scripts/Core/<Category>/I<Category>Loader.cs` **before** the implementation:

```csharp
/// <summary>
/// Contract for async loading and lifecycle management of <Category> assets.
/// Use the generic type T to match the actual asset type (GameObject, AudioClip, Sprite, etc.).
/// </summary>
public interface I<Category>Loader<T> where T : UnityEngine.Object
{
    /// <summary>Loads a single asset by its Addressable address. Returns cached result if already loaded.</summary>
    UniTask<T> LoadAsync(string address, CancellationToken ct = default);

    /// <summary>Loads all assets matching the specified Addressable label.</summary>
    UniTask<IList<T>> LoadAllByLabelAsync(string label, CancellationToken ct = default);

    /// <summary>Releases the handle for the given address. No-op if not loaded.</summary>
    void Release(string address);

    /// <summary>Releases all active handles for the label batch. Call after batch is no longer needed.</summary>
    void ReleaseLabel(string label);
}
```

---

## Step 6 — Create the Loader Implementation

Create `Scripts/Systems/<Category>/<Category>Loader.cs`:

```csharp
/// <summary>
/// Implements <see cref="I<Category>Loader{T}"/> for <Category> assets.
/// Tracks all handles internally to prevent leaks. Must be disposed when the owning scope is destroyed.
/// </summary>
public sealed class <Category>Loader<T> : I<Category>Loader<T>, IDisposable where T : UnityEngine.Object
{
    // Tracks individual asset handles for per-address release
    private readonly Dictionary<string, AsyncOperationHandle<T>> _singleHandles = new();

    // Tracks batch handles for label-based release
    private readonly Dictionary<string, AsyncOperationHandle<IList<T>>> _labelHandles = new();

    /// <inheritdoc/>
    public async UniTask<T> LoadAsync(string address, CancellationToken ct = default)
    {
        // Return cached result immediately if handle is still valid
        if (_singleHandles.TryGetValue(address, out var existing))
            return existing.Result;

        var handle = Addressables.LoadAssetAsync<T>(address);
        _singleHandles[address] = handle;

        try
        {
            return await handle.WithCancellation(ct);
        }
        catch (OperationCanceledException)
        {
            Addressables.Release(handle);
            _singleHandles.Remove(address);
            throw;
        }
    }

    /// <inheritdoc/>
    public async UniTask<IList<T>> LoadAllByLabelAsync(string label, CancellationToken ct = default)
    {
        // Return cached batch result if already loaded
        if (_labelHandles.TryGetValue(label, out var existing))
            return existing.Result;

        // Store the handle so it can be released later — prevents handle leak
        var handle = Addressables.LoadAssetsAsync<T>(label, null);
        _labelHandles[label] = handle;

        try
        {
            return await handle.WithCancellation(ct);
        }
        catch (OperationCanceledException)
        {
            Addressables.Release(handle);
            _labelHandles.Remove(label);
            throw;
        }
    }

    /// <inheritdoc/>
    public void Release(string address)
    {
        if (_singleHandles.TryGetValue(address, out var handle))
        {
            Addressables.Release(handle);
            _singleHandles.Remove(address);
        }
    }

    /// <inheritdoc/>
    public void ReleaseLabel(string label)
    {
        if (_labelHandles.TryGetValue(label, out var handle))
        {
            Addressables.Release(handle);
            _labelHandles.Remove(label);
        }
    }

    /// <summary>
    /// Releases all active handles — both single and batch.
    /// Call via IDisposable when the owning scene or scope is destroyed.
    /// </summary>
    public void Dispose()
    {
        foreach (var handle in _singleHandles.Values)
            Addressables.Release(handle);

        foreach (var handle in _labelHandles.Values)
            Addressables.Release(handle);

        _singleHandles.Clear();
        _labelHandles.Clear();
    }
}
```

---

## Step 7 — Wire into GameBootstrapper

In `GameBootstrapper.cs` (or the relevant scene Bootstrapper):

```csharp
// Example: loader for GameObject Prefabs
private I<Category>Loader<GameObject> _<category>Loader;

private void Awake()
{
    _<category>Loader = new <Category>Loader<GameObject>();
    _systemThatNeedsAssets.Construct(_<category>Loader);
}

private void OnDestroy()
{
    (_<category>Loader as IDisposable)?.Dispose();
}
```

> For non-Prefab assets, substitute the type: `<Category>Loader<AudioClip>`, `<Category>Loader<Sprite>`, `<Category>Loader<MyScriptableObject>`, etc.

---

## Step 8 — Verify No Remaining Resources.Load Calls

```bash
grep -rn "Resources\.Load" Assets/Scripts --include="*.cs"
```

Any remaining hits → migrate to the new loader.

---

## Step 9 — Build Addressables & Test

1. Open **Window → Asset Management → Addressables → Groups**
2. Click **Build → New Build → Default Build Script**
3. Enter Play Mode — trigger a loading call and verify the asset appears
4. Open **Window → Asset Management → Addressables → Event Viewer**

**Verification checklist:**
- [ ] Asset loads without errors in console
- [ ] Event Viewer shows handle count increasing on load, decreasing on release
- [ ] Handle count returns to 0 after `Dispose()` is called (scene unload test)
- [ ] Batch load (`LoadAllByLabelAsync`) releases correctly via `ReleaseLabel()`
- [ ] Cancellation mid-load does not leave a leaked handle (cancel test)

---

## Step 9A — Pre-warm Pattern (If "at boot" from Step 1)

For assets that must be available immediately at scene start, pre-load during the loading screen:

```csharp
// In LoadingState or GameplayBootstrapper, before activating the scene:
private async UniTask PreWarmAssetsAsync(CancellationToken ct)
{
    // Pre-load critical assets before revealing the gameplay scene
    await _<category>Loader.LoadAllByLabelAsync("<category>", ct);
    // Assets are now cached — zero load latency during gameplay
}
```

Wire this into `LoadingState.OnEnter()` before `OnLoadComplete` fires.

---

## Step 10 — Commit

```bash
git add Assets/AddressableAssetsData/
git commit -m "chore(addressables): add <GroupName> group with <N> assets and labels"

git add Scripts/Core/<Category>/ Scripts/Systems/<Category>/
git commit -m "feat(<category>): add generic <Category>Loader<T> with handle leak prevention"
```

---

## Cross-Skill References
- Addressables patterns and memory management → `skills/asset-loading`
- Interface + implementation structure → `skills/create-feature`
- Handle lifecycle profiling and leak detection → `skills/unity-profiler-mind`
