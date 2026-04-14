---
name: scene-management
description: Use when architecting scene loading, transitions, additive scene workflows, or bootstrapping system dependencies per-scene in Unity.
---

# Skill: Scene Management & Bootstrapping

---

## Core Rules

| Rule | Requirement |
|---|---|
| `SceneManager.LoadScene()` (synchronous) | **Forbidden** during gameplay — causes multi-frame freeze on mobile |
| `DontDestroyOnLoad` abuse | Restricted — only for services that genuinely must persist (Audio, Analytics) |
| Scene coupling | Scenes must never have direct GameObject references to other scenes |
| Bootstrapping | Every scene must have a self-contained `Bootstrapper` (Manual DI default) or `LifetimeScope` (VContainer, if adopted) |

---

## Scene Architecture

### Scene Types & Responsibilities

```
[Persistent Scene]    — Loaded once at app launch, never unloaded
   ├── Core Services (AudioManager, AnalyticsService, InputSystem)
   ├── PersistentBootstrapper (Manual DI root — creates and wires all global services)
   └── SceneLoader (manages all other scene transitions)

[Menu Scene]          — Loaded additively over Persistent
   ├── MainMenuController
   └── MenuBootstrapper (receives global services via constructor injection)

[Gameplay Scene]      — Loaded additively, Menu unloaded
   ├── GameplayBootstrapper (scene-scoped DI root)
   ├── World/Level content
   └── (global services passed from PersistentBootstrapper via SceneLoader)

[UI Overlay Scene]    — Optionally additive (HUD, Pause screen)
   └── HUDBootstrapper (receives IAudioService, IInputProvider, etc.)
```

---

## Async Loading Pattern

### Standard Scene Transition Flow

```csharp
// SceneLoader.cs — registered as singleton in global LifetimeScope
public sealed class SceneLoader : ISceneLoader
{
    private AsyncOperationHandle<SceneInstance> _currentSceneHandle;

    public event Action<float> OnLoadProgress;
    public event Action OnLoadComplete;

    public async UniTask LoadSceneAsync(
        string sceneAddress,
        bool unloadCurrent = true,
        IProgress<float> progress = null,
        CancellationToken ct = default)
    {
        // 1. Optional: fire loading screen event
        // 2. Unload current gameplay scene
        if (unloadCurrent && _currentSceneHandle.IsValid())
            await Addressables.UnloadSceneAsync(_currentSceneHandle).Task.AttachExternalCancellation(ct);

        // 3. Load new scene additively
        _currentSceneHandle = Addressables.LoadSceneAsync(sceneAddress, LoadSceneMode.Additive);

        while (!_currentSceneHandle.IsDone)
        {
            progress?.Report(_currentSceneHandle.PercentComplete);
            OnLoadProgress?.Invoke(_currentSceneHandle.PercentComplete);
            await UniTask.Yield(ct);
        }

        if (_currentSceneHandle.Status != AsyncOperationStatus.Succeeded)
            throw new Exception($"[SceneLoader] Failed to load scene: {sceneAddress}");

        // 4. Activate the new scene
        SceneManager.SetActiveScene(_currentSceneHandle.Result.Scene);
        OnLoadComplete?.Invoke();
    }
}
```

### Loading Screen Integration

```csharp
// LoadingScreenView.cs — reacts to ISceneLoader events
public sealed class LoadingScreenView : MonoBehaviour
{
    [SerializeField] private Slider _progressBar;
    [SerializeField] private CanvasGroup _canvasGroup;

    private ISceneLoader _sceneLoader;

    public void Construct(ISceneLoader sceneLoader)
    {
        _sceneLoader = sceneLoader;
        _sceneLoader.OnLoadProgress += UpdateProgress;
        _sceneLoader.OnLoadComplete += Hide;
    }

    private void Show() => _canvasGroup.alpha = 1f;
    private void Hide() => _canvasGroup.alpha = 0f;
    private void UpdateProgress(float value) => _progressBar.value = value;

    private void OnDestroy()
    {
        if (_sceneLoader == null) return;
        _sceneLoader.OnLoadProgress -= UpdateProgress;
        _sceneLoader.OnLoadComplete -= Hide;
    }
}
```

---

## Scene Bootstrapping Pattern

Each scene must initialize its own dependency tree without relying on other scenes' objects.

```csharp
// GameplayBootstrapper.cs — Manual DI root for this scene (default pattern)
public sealed class GameplayBootstrapper : MonoBehaviour
{
    [SerializeField] private GameplayConfig _config;
    [SerializeField] private PlayerView _playerView;

    private void Awake()
    {
        // 1. Create pure C# systems
        IHealthSystem health = new HealthSystem(_config.PlayerMaxHealth);
        IInventorySystem inventory = new InventorySystem(_config.InventoryCapacity);

        // 2. Inject into MonoBehaviour views via Construct()
        _playerView.Construct(health, inventory);
    }
}
```

> **Optional upgrade — VContainer**: If the scene has a large number of dependencies (> 8 systems) or requires child scopes, introduce `LifetimeScope` from VContainer. Consult `skills/create-feature` (Step 5) for the migration decision framework.

```csharp
// GameplayLifetimeScope.cs — VContainer alternative (adopt only when Manual DI becomes unwieldy)
public sealed class GameplayLifetimeScope : LifetimeScope
{
    [SerializeField] private GameplayConfig _config;

    protected override void Configure(IContainerBuilder builder)
    {
        builder.Register<HealthSystem>(Lifetime.Scoped).As<IHealthSystem>();
        builder.Register<InventorySystem>(Lifetime.Scoped).As<IInventorySystem>();
        builder.RegisterInstance(_config);
        builder.RegisterComponentInHierarchy<PlayerView>();
    }
}
```

---

## DontDestroyOnLoad — Allowed vs Forbidden

| Service | DontDestroyOnLoad? | Reason |
|---|---|---|
| AudioManager | ✅ Allowed | Music must persist across transitions |
| AnalyticsService | ✅ Allowed | Session tracking must not reset |
| InputSystem | ✅ Allowed | Input mapping persists all scenes |
| GameManager / UIManager | ❌ Forbidden | Re-instantiate per scene via Bootstrapper |
| PlayerController | ❌ Forbidden | Scene-scoped, dispose on unload |
| EnemyManager | ❌ Forbidden | Scene-scoped |

---

## Scene-Level Memory Cleanup
When a scene is unloaded, ensure:
1. All Addressable handles loaded by that scene are released
2. All event subscriptions scoped to that scene are cleaned up
3. VContainer `LifetimeScope` is disposed (automatic if using VContainer correctly)
4. Call `Resources.UnloadUnusedAssets()` + `GC.Collect()` after major scene transitions (not during gameplay)

```csharp
private async UniTask OnSceneUnloaded(CancellationToken ct)
{
    await Resources.UnloadUnusedAssets().ToUniTask(cancellationToken: ct);
    GC.Collect();
}
```

---

## Scene Transition State Machine

```
AppLaunch
    └── [Load Persistent Scene]
            └── [Load Splash/Boot Scene]
                    └── [Show Loading Screen]
                            ├── [Load Menu Scene] ←──────────────────────────┐
                            │       └── [User starts game]                   │
                            └── [Load Gameplay Scene]                        │
                                    ├── [Game Over] ──► [Unload Gameplay] ───┘
                                    └── [Victory]  ──► [Unload Gameplay] ───┘
```

---

## Cross-Skill References
- Loading scene assets via Addressables → `skills/asset-loading`
- Cleaning up unmanaged resources in scene services → `skills/unity-profiler-mind` (Memory Leak section)
- Per-scene DI root configuration → `skills/create-feature` (Step 5)
- Audio persistence across scenes → `skills/audio-architecture`
