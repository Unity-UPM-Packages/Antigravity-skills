---
name: dev-modular-design
description: Use when architecting C# components, reviewing component responsibilities, or designing how systems communicate. Also activates proactively when the AI detects a MonoBehaviour doing multiple unrelated things, direct component references instead of interfaces, or missing event-based decoupling — at any point during a task.
---

# Skill: Modular Design Thinking

---

## Core Principles

### 1. Decoupling First
A component must function independently without crashing if surrounding optional systems are disconnected.

**Self-check test**: "If I deleted every other script, would this one still compile and initialize safely?"
- If NO → it has an illegal hard dependency. Inject via interface instead.

```csharp
// ❌ Hard dependency — fragile
public sealed class PlayerController : MonoBehaviour
{
    private void Awake() => _ui = FindObjectOfType<HUDManager>(); // Breaks in isolation
}

// ✅ Decoupled — injected contract
public sealed class PlayerController : MonoBehaviour
{
    [SerializeField] private PlayerView _view; // Only own-layer dependency
    private IHealthSystem _health;
    public void Construct(IHealthSystem health) => _health = health;
}
```

### 2. Composition Over Inheritance
Favor attaching small, focused components over deep inheritance hierarchies. Deep hierarchies are brittle — a change to the base propagates unpredictably.

```csharp
// ❌ Inheritance-heavy (fragile, rigid)
public class SwimmingFlyingShootingEnemy : FlyingEnemy { }

// ✅ Composition (extensible, replaceable)
[RequireComponent(typeof(MovementComponent))]
[RequireComponent(typeof(ShootingComponent))]
[RequireComponent(typeof(HealthComponent))]
public sealed class EnemyController : MonoBehaviour { }
```

### 3. Interface Contracts Before Concrete Classes
Always define behavior as an interface **before** writing implementation. This enforces Dependency Inversion and makes unit testing possible without scene setup.

```csharp
public interface IHealthSystem
{
    int CurrentHealth { get; }
    int MaxHealth { get; }
    void TakeDamage(int amount);
    void Heal(int amount);
    event Action<int> OnHealthChanged;
    event Action OnDied;
}

public sealed class HealthSystem : IHealthSystem { ... }
```

### 4. Event-Driven Communication
Systems notify others via events — never by directly calling methods on concrete references.

| Channel Type | When to Use | Mechanism |
|---|---|---|
| One-to-one callback | Same system, tight scope | `Action<T>` / `Func<T>` |
| Domain event | Cross-system notifications | `event Action<T>` on interface |
| Global broadcast | Fully decoupled N:N | `EventBus<TEvent>` (static or DI) |
| Reactive stream | Sequences / throttle / transform | UniRx / R3 `Observable` |

**Rule**: If System A calls a method on System B directly, they are coupled. Replace with an event System A fires; System B listens.

### 5. Assembly Definition Isolation
Group related systems into `.asmdef` files to enforce hard compilation boundaries and dependency direction:

```
Game.Core.asmdef         ← Pure C# logic, zero Unity dependency
Game.Gameplay.asmdef     ← MonoBehaviour wrappers, depends on Core
Game.UI.asmdef           ← UI layer, depends on Core ONLY (not Gameplay directly)
Game.Audio.asmdef        ← Audio system, depends on Core
Game.Tests.Editor.asmdef ← EditMode tests, references Core + Gameplay
```

This ensures UI can never accidentally call gameplay logic directly, and Core logic remains engine-agnostic.

### 6. Lifecycle Cleanup Contract
Every component that subscribes to an event **must** dispose/unsubscribe in `OnDestroy`. Failure causes phantom callbacks and memory leaks.

```csharp
private void OnEnable() => _health.OnDied += HandleDeath;
private void OnDisable() => _health.OnDied -= HandleDeath;
```

---

## When NOT to Decouple (Over-Engineering Warning)

❌ Do NOT introduce interfaces + event buses for:
- Internal utility classes used in exactly one place
- Systems that will realistically never have a second implementation
- Early-prototype code still changing shape every day (label with `// PROTOTYPE — refactor before merge`)

The cost of premature abstraction is real. Only abstract what genuinely needs to vary.

---

## Modular Design Checklist
Before committing any new component:
- [ ] The class has exactly ONE primary responsibility (no "and" in its description)
- [ ] It depends on interfaces, not concrete classes (except MonoBehaviour lifecycle)
- [ ] It can be instantiated in a unit test without a running scene
- [ ] It communicates changes outward via events, not by calling other systems directly
- [ ] All event subscriptions are cleaned up in `OnDisable` or `Dispose`
- [ ] It is in the correct `.asmdef` assembly for its layer

---

## Cross-Skill References
- For pattern selection when decoupling complex systems → `skills/design-pattern-architect`
- For implementing a new fully modular feature → `skills/create-feature`
- For testing components in isolation → `skills/test-driven-dev`
