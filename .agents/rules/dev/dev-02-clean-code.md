---
trigger: glob
glob: "*.cs"
description: "Clean Code principles — SOLID, DRY, KISS, YAGNI, error handling strategy, method/class size limits, dead code prohibition, and cyclomatic complexity control."
---

# Rule 02: Clean Code & SOLID Principles

## Overview
Every line of code must be readable, maintainable, and intentional. Code is read 10× more than it is written — optimize for the reader, not the writer. This rule combines Uncle Bob's Clean Code philosophy with SOLID principles adapted for Unity/C#.

---

## 1. SOLID Principles

### Single Responsibility (SRP)
Every class must have **only ONE reason to change**.
- ❌ `PlayerController` that handles input, movement, health, UI updates, and audio
- ✅ `PlayerInput`, `PlayerMovement`, `HealthSystem`, `HUDPresenter`, `AudioService` — each with one job

### Open/Closed (OCP)
Classes should be **open for extension, closed for modification**.
- Use interfaces + new implementations instead of modifying existing classes
- Use Strategy pattern or ScriptableObject configs instead of adding `if/else` branches

### Liskov Substitution (LSP)
Subtypes must be substitutable for their base types without breaking behavior.
- If `Fireball : Spell` overrides `Cast()` and crashes because it needs `_manaSystem` that `Spell` doesn't require → LSP violation

### Interface Segregation (ISP)
Clients should not depend on interfaces they don't use.
- ❌ `IEntity { void Move(); void Shoot(); void Fly(); }` — not all entities fly
- ✅ `IMovable { void Move(); }` + `IShootable { void Shoot(); }` + `IFlyable { void Fly(); }`

### Dependency Inversion (DIP)
High-level modules must not depend on low-level modules. Both depend on abstractions.
- ❌ `PlayerController` directly references `AudioManager` concrete class
- ✅ `PlayerController` receives `IAudioService` via constructor injection

---

## 2. Core Clean Code Principles

### DRY — Don't Repeat Yourself
If the same logic appears in 2+ places, extract it:
- Identical code blocks → Extract to a shared method
- Similar classes with slight variations → Extract base class or use Strategy
- Repeated magic numbers → Extract to `const` or `[SerializeField]`

**Exception**: Don't force DRY on coincidentally similar code that may diverge later. Premature abstraction is worse than duplication.

### KISS — Keep It Simple, Stupid
Always choose the simplest solution that works:
- ❌ Generic AbstractFactoryProvider<T> for a system used in exactly one place
- ✅ Simple class with a `Create()` method
- If you can't explain your design in one sentence, it's too complex

### YAGNI — You Aren't Gonna Need It
Never build features or abstractions "just in case":
- ❌ Creating an event bus, plugin system, and 3 interfaces for a feature that currently has one implementation
- ✅ Build the simplest version → refactor when a second use case actually appears
- Label speculative code: `// YAGNI — revisit if needed`

---

## 3. Method & Class Size Limits

### Methods
| Guideline | Limit |
|---|---|
| Method length | **≤ 20 lines** ideal, **≤ 30 lines** max |
| Parameters | **≤ 3 parameters** (use object/struct for more) |
| Nesting depth | **≤ 2 levels** of if/for/while nesting |
| Return points | Use **early returns** (guard clauses) to reduce nesting |

```csharp
// ❌ Deep nesting
public void Process(Order order)
{
    if (order != null)
    {
        if (order.IsValid)
        {
            if (order.HasItems)
            {
                // actual logic buried 3 levels deep
            }
        }
    }
}

// ✅ Guard clauses — flat and readable
public void Process(Order order)
{
    if (order == null) return;
    if (!order.IsValid) return;
    if (!order.HasItems) return;

    // actual logic at root level
}
```

### Classes
| Guideline | Limit |
|---|---|
| Class length | **≤ 200 lines** ideal, **≤ 300 lines** warning zone |
| Responsibilities | **Exactly 1** (SRP) |
| Dependencies | **≤ 4 injected interfaces** — more = likely doing too much |

If a class exceeds 200 lines, audit whether it violates SRP and extract sub-components.

---

## 4. Error Handling Strategy

### Never Return Null
Null returns force every caller to add null checks — a maintenance nightmare.

```csharp
// ❌ Returns null — caller must check
public IWeapon GetWeapon(string id)
{
    return _weapons.ContainsKey(id) ? _weapons[id] : null;
}

// ✅ Option A: Return a NullObject
public IWeapon GetWeapon(string id)
{
    return _weapons.ContainsKey(id) ? _weapons[id] : NullWeapon.Instance;
}

// ✅ Option B: Use TryGet pattern
public bool TryGetWeapon(string id, out IWeapon weapon)
{
    return _weapons.TryGetValue(id, out weapon);
}
```

### Exception Strategy
| Situation | Approach |
|---|---|
| Programmer error (null arg, invalid state) | `throw ArgumentNullException` / `InvalidOperationException` |
| Expected failure (file not found, network error) | Return `Result<T>` or `bool TryX()` pattern |
| Unity lifecycle (destroyed object) | Guard with `if (this == null)` or null-conditional `?.` |

### Guard Clauses at Method Entry
```csharp
public void TakeDamage(int amount, IDamageSource source)
{
    if (amount <= 0) return;                           // Invalid input
    if (_isDead) return;                                // Invalid state
    if (source == null) throw new ArgumentNullException(nameof(source));
    
    // Core logic starts here — clean and confident
}
```

---

## 5. Dead Code Prohibition

### Rules
- **Never commit commented-out code** — use version control (`git stash`, feature branches)
- **Never leave unused `using` directives** — IDE auto-cleans these
- **Never leave empty methods** (`Update() { }`) — delete or add `// TODO` with justification
- **Flag TODO/HACK**: All temporary code must carry `// TODO:` or `// HACK:` tags with explanation and ticket reference

### Detection Signals
If you see any of these, remove immediately:
- `// old implementation`
- `#if false ... #endif`
- Methods not called anywhere
- Parameters never used
- Variables assigned but never read

---

## 6. Cyclomatic Complexity Control

| Complexity | Rating | Action |
|---|---|---|
| 1–5 | ✅ Simple | Good |
| 6–10 | ⚠️ Moderate | Review — consider extracting |
| 11–20 | 🔴 Complex | Refactor mandatory — extract methods or use patterns |
| 20+ | 💀 Unmaintainable | Rewrite with Strategy/State pattern |

### Reduction Techniques
- Replace `switch` on enum with **polymorphism** or **dictionary dispatch**
- Replace nested `if/else` with **guard clauses**
- Replace complex boolean expressions with **named boolean methods**

```csharp
// ❌ Complex condition
if (player.Health > 0 && !player.IsStunned && player.Mana >= spell.Cost && !isCooldown)

// ✅ Named conditions
if (CanCastSpell(player, spell))

private bool CanCastSpell(Player player, Spell spell)
{
    return player.IsAlive && !player.IsStunned && player.HasEnoughMana(spell.Cost) && !spell.IsOnCooldown;
}
```

---

## 7. Script Composition (Unity-Specific)

### MonoBehaviour Discipline
- Restrict `MonoBehaviour` to the **View layer** (visuals, Unity lifecycle, collisions)
- Pure Logic, Data Models, and Services MUST be **plain C# classes** unrelated to engine lifecycle
- If a script exceeds 150 lines, it likely violates SRP — split into components

### Prefabs as Legos
- Scripts should not break if placed on entirely different Prefabs without sibling components
- Use `[RequireComponent]` for hard dependencies, interfaces for soft dependencies
- Favor `Construct()` injection over `GetComponent<T>()` in `Awake()`

### Interface-First Design
Always define behavior as an interface BEFORE writing implementation:
```csharp
public interface IHealthSystem
{
    int CurrentHealth { get; }
    void TakeDamage(int amount);
    event Action OnDied;
}

public sealed class HealthSystem : IHealthSystem { ... }
```
