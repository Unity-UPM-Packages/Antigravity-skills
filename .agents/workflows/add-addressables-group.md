# Workflow: Add Addressables Group

Add a new Addressables Group to manage a specific category of assets (e.g., Enemy Prefabs, Level Environments, UI Bundles, Audio Clips). Ensures runtime loading is correct, labeled, backed by a loader script, and free of any `Resources.Load` calls.

---

## Step 1 — Define the Group

Answer before starting:
```
1. What type of assets does this group contain?
   (Prefabs / Textures / Audio / ScriptableObjects / Scenes)

2. When are they loaded?
   (at boot / on level start / on-demand during gameplay)

3. Should assets be released after use?
   (e.g., enemy dies → release handle)

4. Approximate asset count?
   (< 10 / 10–100 / > 100)
```

---

## Step 2 — Create the Addressables Group

Open **Window → Asset Management → Addressables → Groups**:

1. Click `Create → Group → Packed Assets`
2. Name the group: `[Category]_[Type]` (e.g., `Enemies_Prefabs`, `UI_Backgrounds`, `Audio_Music`)
3. Configure Group Settings:
   - `Build Path`: `[UnityEngine.AddressableAssets.Addressables.BuildPath]`
   - `Load Path`: `[UnityEngine.AddressableAssets.Addressables.RuntimeLoadPath]`
   - `Bundle Mode`: Pack Together (assets usually loaded together) or Pack Separately (loaded independently)

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

## Step 5 — Create the Loader Script

Create `Scripts/Systems/<Category>/<Category>Loader.cs`:

```csharp
/// <summary>
/// Provides async loading and lifecycle management for <Category> assets via Addressables.
/// Implements <see cref="I<Category>Loader"/> and <see cref="IDisposable"/>.
/// </summary>
public sealed class <Category>Loader : I<Category>Loader, IDisposable
{
    private readonly Dictionary<string, AsyncOperationHandle<GameObject>> _handles = new();

    /// <inheritdoc/>
    public async UniTask<GameObject> LoadAsync(string address, CancellationToken ct = default)
    {
        if (_handles.TryGetValue(address, out var existing))
            return existing.Result;

        var handle = Addressables.LoadAssetAsync<GameObject>(address);
        _handles[address] = handle;

        try
        {
            return await handle.WithCancellation(ct);
        }
        catch (OperationCanceledException)
        {
            Addressables.Release(handle);
            _handles.Remove(address);
            throw;
        }
    }

    /// <inheritdoc/>
    public async UniTask<IList<GameObject>> LoadAllByLabelAsync(string label, CancellationToken ct = default)
    {
        var handle = Addressables.LoadAssetsAsync<GameObject>(label, null);
        return await handle.WithCancellation(ct);
    }

    /// <inheritdoc/>
    public void Release(string address)
    {
        if (_handles.TryGetValue(address, out var handle))
        {
            Addressables.Release(handle);
            _handles.Remove(address);
        }
    }

    /// <summary>Releases all active handles. Call when the owning scope is destroyed (e.g., level unload).</summary>
    public void Dispose()
    {
        foreach (var handle in _handles.Values)
            Addressables.Release(handle);

        _handles.Clear();
    }
}
```

---

## Step 6 — Create the Interface

Create `Scripts/Core/<Category>/I<Category>Loader.cs`:

```csharp
/// <summary>Contract for async loading and release of <Category> assets.</summary>
public interface I<Category>Loader
{
    /// <summary>Loads a single asset by its Addressable address.</summary>
    UniTask<GameObject> LoadAsync(string address, CancellationToken ct = default);

    /// <summary>Loads all assets matching the specified Addressable label.</summary>
    UniTask<IList<GameObject>> LoadAllByLabelAsync(string label, CancellationToken ct = default);

    /// <summary>Releases the handle for the given address.</summary>
    void Release(string address);
}
```

---

## Step 7 — Wire into GameBootstrapper

In `GameBootstrapper.cs`:

```csharp
private I<Category>Loader _<category>Loader;

private void Awake()
{
    _<category>Loader = new <Category>Loader();
    _systemThatNeedsAssets.Construct(_<category>Loader);
}

private void OnDestroy()
{
    (_<category>Loader as IDisposable)?.Dispose();
}
```

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
3. Enter Play Mode in the Editor — verify assets load correctly
4. Open **Window → Asset Management → Addressables → Event Viewer** — confirm no handle leaks

---

## Step 10 — Commit

```bash
git add Assets/AddressableAssetsData/
git commit -m "chore(addressables): add <GroupName> group with <N> assets and labels"

git add Scripts/Core/<Category>/ Scripts/Systems/<Category>/
git commit -m "feat(<category>): add <Category>Loader with async load and release"
```

---

## Cross-Skill References
- Addressables patterns and memory management → `skills/asset-loading`
- Interface + implementation structure → `skills/create-feature`
- Handle lifecycle profiling → `skills/unity-profiler-mind`
