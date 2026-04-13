---
name: create-feature
description: Use when implementing any new system, mechanic, functionality, or behaviour in the project — whether the user says "create", "add", "implement", "build", "I want X to happen", or describes desired behaviour. Also activates proactively when the AI determines a new independent system is needed to support a request. Enforces interface-first, modular, and UPM-extractable architecture on every implementation.
---

# Workflow: Feature Implementation

## Objective
Establish a strict Standard Operating Procedure for building any new gameplay system. Features built outside this workflow accumulate technical debt immediately and become candidates for painful rewrites within 2–3 sprints.

---

## Pre-Implementation: Scope Gate
Before writing a single line of code, answer these 4 questions:
1. **Single Responsibility**: What is the ONE thing this feature does? If the answer contains "and", split into sub-features.
2. **Data Boundary**: What data does it read? What data does it write? Who owns that data?
3. **Event Surface**: Who needs to know when something happens in this system?
4. **Duplication Check**: Does a similar system already exist in the codebase? Extend it rather than duplicate.

---

## Execution Sequence

### Step 1 — Define Contracts (Interfaces)
Create interface files in `Scripts/Core/<FeatureName>/`. No implementation yet. This step forces clarity about what the system actually does before getting lost in implementation details.

```csharp
// IInventorySystem.cs — in Game.Core.asmdef
public interface IInventorySystem
{
    bool TryAddItem(ItemData item);
    bool TryRemoveItem(string itemId);
    IReadOnlyList<ItemData> GetAllItems();
    event Action<ItemData> OnItemAdded;
    event Action<ItemData> OnItemRemoved;
}
```

### Step 2 — Define Data Models
Create plain C# structs or classes (zero `MonoBehaviour`) for all data containers. Prefer `readonly struct` for value objects that are passed frequently.

```csharp
// ItemData.cs — in Game.Core.asmdef
[Serializable]
public readonly struct ItemData
{
    public string Id { get; }
    public string DisplayName { get; }
    public Sprite Icon { get; }      // Note: Sprite ref OK for data bag; don't put logic here
    public int Quantity { get; }

    public ItemData(string id, string displayName, Sprite icon, int quantity)
    {
        Id = id;
        DisplayName = displayName;
        Icon = icon;
        Quantity = quantity;
    }
}
```

### Step 3 — Implement Pure Logic
Write the concrete implementation as plain C# in `Scripts/Systems/<FeatureName>/`. No engine coupling, no `MonoBehaviour`, no `UnityEngine` namespace imports beyond data types.

```csharp
// InventorySystem.cs — in Game.Gameplay.asmdef
public sealed class InventorySystem : IInventorySystem
{
    private readonly List<ItemData> _items = new();

    public event Action<ItemData> OnItemAdded;
    public event Action<ItemData> OnItemRemoved;

    public bool TryAddItem(ItemData item)
    {
        _items.Add(item);
        OnItemAdded?.Invoke(item);
        return true;
    }

    public IReadOnlyList<ItemData> GetAllItems() => _items;
    // ...
}
```

### Step 4 — Create MonoBehaviour Bridge (View Layer)
Wrap the pure logic in a thin MonoBehaviour in `Scripts/Views/<FeatureName>/` that handles:
- Unity lifecycle wiring (`Awake` init, `OnDestroy` cleanup)
- `[SerializeField]` config value exposure to the Inspector
- Delegating visual feedback (animations, particles, audio) — never logic

```csharp
// InventoryView.cs — in Game.Gameplay.asmdef
[RequireComponent(typeof(InventoryAnimator))]
public sealed class InventoryView : MonoBehaviour
{
    [SerializeField] private InventoryConfig _config;

    private IInventorySystem _inventory;

    // Called by DI framework or Bootstrapper
    public void Construct(IInventorySystem inventory)
    {
        _inventory = inventory;
        _inventory.OnItemAdded += HandleItemAdded;
    }

    private void OnDestroy()
    {
        if (_inventory != null)
            _inventory.OnItemAdded -= HandleItemAdded;
    }

    private void HandleItemAdded(ItemData item)
    {
        // Trigger visual feedback only — no business logic here
        GetComponent<InventoryAnimator>().PlayAddAnimation(item);
    }
}
```

### Step 5 — Wire Dependencies (Composition Root)

Dependency injection does NOT require a framework. The three approaches below are ordered from **simplest to most powerful**. Start with Tier 1 and migrate up only when genuine pain is felt.

#### Tier 1 — Manual DI via Bootstrapper ✅ Default (Recommended)
Create a `Bootstrapper` MonoBehaviour in the scene that manually constructs and wires all systems. This is framework-free, debuggable, and fully UPM-package-compatible.

```csharp
// GameBootstrapper.cs — one per scene, executed first (Script Execution Order)
public sealed class GameBootstrapper : MonoBehaviour
{
    [Header("Config")]
    [SerializeField] private InventoryConfig _inventoryConfig;

    [Header("Views")]
    [SerializeField] private InventoryView _inventoryView;
    [SerializeField] private PlayerView _playerView;

    private IInventorySystem _inventory;

    private void Awake()
    {
        // 1. Create pure logic systems (no MonoBehaviour needed)
        _inventory = new InventorySystem(_inventoryConfig.Capacity);

        // 2. Inject into Views via Construct() — no FindObjectOfType, no Singleton
        _inventoryView.Construct(_inventory);
        _playerView.Construct(_inventory);
    }

    private void OnDestroy()
    {
        // Explicit cleanup if systems implement IDisposable
        (_inventory as IDisposable)?.Dispose();
    }
}
```

**Why this works for UPM**: The `InventorySystem` and its interface live in the package. The `GameBootstrapper` lives in the project. Zero dependency leaks.

#### Tier 2 — ServiceLocator (Cross-scene lookup)
Use when a system created in Scene A must be found by a MonoBehaviour added to Scene B (e.g., persistent AudioService). Keep it small and typed to interfaces.

```csharp
// ServiceLocator.cs — a simple static registry
public static class ServiceLocator
{
    private static readonly Dictionary<Type, object> _services = new();

    public static void Register<T>(T service) where T : class
        => _services[typeof(T)] = service;

    public static T Get<T>() where T : class
    {
        if (_services.TryGetValue(typeof(T), out object service))
            return (T)service;

        Debug.LogError($"[ServiceLocator] Service not registered: {typeof(T).Name}");
        return null;
    }

    public static void Unregister<T>() where T : class
        => _services.Remove(typeof(T));
}

// Usage in persisted services:
public sealed class AudioBootstrapper : MonoBehaviour
{
    private void Awake() => ServiceLocator.Register<IAudioService>(new AudioService(...));
    private void OnDestroy() => ServiceLocator.Unregister<IAudioService>();
}
```

#### Tier 3 — VContainer (Optional upgrade path)
Migrate to VContainer only when the Bootstrapper grows unwieldy (typically 15+ systems with complex lifetime management). The migration is mechanical — interfaces and systems don't change, only the wiring file does.

```csharp
// GameLifetimeScope.cs — VContainer (optional, future migration)
public sealed class GameLifetimeScope : LifetimeScope
{
    protected override void Configure(IContainerBuilder builder)
    {
        builder.Register<InventorySystem>(Lifetime.Singleton).As<IInventorySystem>();
        builder.RegisterComponentInHierarchy<InventoryView>();
    }
}
```

> Since all systems depend on interfaces only, switching from Manual DI → VContainer requires changing only the Bootstrapper file. All feature code stays identical.

### Step 6 — Tests & Integration Report
- Write at minimum **2 EditMode tests**: one happy-path, one edge-case (null input, empty state, max capacity).
- Invoke `skills/test-driven-dev` for the full TDD workflow.
- Present a brief integration summary: systems added, events emitted, how other systems can subscribe.

---

## Folder Structure Convention

### Standard (Single Project)
```
Scripts/
├── Core/                         ← Game.Core.asmdef (no Unity/framework deps)
│   └── Inventory/
│       ├── IInventorySystem.cs   ← Interface contract
│       └── ItemData.cs           ← Data model (plain C# struct)
├── Systems/                      ← Game.Gameplay.asmdef
│   └── Inventory/
│       ├── InventorySystem.cs    ← Pure logic implementation
│       └── InventoryConfig.cs    ← ScriptableObject config
├── Views/                        ← Game.Gameplay.asmdef
│   └── Inventory/
│       ├── InventoryView.cs      ← MonoBehaviour bridge
│       └── InventoryAnimator.cs  ← Visual-only component
└── Bootstrap/                    ← Game.Bootstrap.asmdef
    └── GameBootstrapper.cs       ← Wiring only — not in package
```

### UPM Package Structure (when extracted)
```
Packages/com.yourname.inventory/
├── Runtime/
│   ├── IInventorySystem.cs       ← Core interface
│   ├── ItemData.cs               ← Data model
│   ├── InventorySystem.cs        ← Implementation
│   ├── InventoryConfig.cs        ← ScriptableObject
│   └── com.yourname.inventory.Runtime.asmdef
├── Editor/
│   └── com.yourname.inventory.Editor.asmdef
├── Tests/
│   └── EditMode/
│       └── InventorySystemTests.cs
└── package.json
```

**Key rule for UPM readiness**: The `Runtime/` assembly must have **zero dependencies** on framework packages (VContainer, UniRx, etc.). If a consumer project uses VContainer, they write their own `LifetimeScope` to register your `InventorySystem`. Your package just provides the interface and implementation.

---

## Anti-Patterns to Reject Immediately
| ❌ Anti-Pattern | ✅ Correct Approach |
|---|---|
| `public` fields on MonoBehaviour | `[SerializeField] private` |
| Logic inside `Update()` that could be event-driven | Subscribe to events in `OnEnable` |
| `FindObjectOfType<T>()` to locate dependencies | Inject via DI or `Construct()` |
| Skipping the interface step for "simple features" | Always define the interface — simplicity changes |
| Placing network/save calls inside the view | Delegate to a service via interface |

---

## Cross-Skill References
- For deciding system structure before coding → `skills/design-pattern-architect`
- For component decoupling principles → `skills/modular-design`
- For writing tests alongside the feature → `skills/test-driven-dev`
- For dynamic asset loading inside the feature → `skills/asset-loading`
