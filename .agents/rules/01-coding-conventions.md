# Rule 01: C# & Unity Coding Conventions

## Overview
A strict standardization for naming conventions, code formatting, documentation, and structure syntax to maintain absolute readability across the entire Unity project. Adherence to this file guarantees a uniform codebase regardless of which developer or AI generates the code.

## 1. Naming Conventions
- **Classes, Structs, Enums**: `PascalCase` (e.g., `GameManager`, `InventoryItem`).
- **Interfaces**: `PascalCase` prefixed strictly with an `I` (e.g., `IDamageable`, `IInteractable`).
- **Public Methods & Properties**: `PascalCase` (e.g., `TakeDamage()`, `CurrentHealth`).
- **Private & Protected Fields**: `camelCase` prefixed with an underscore `_` (e.g., `_currentHealth`, `_playerSpeed`). Absolutely DO NOT use the archaic `m_` prefix.
- **Local Variables & Parameters**: `camelCase` (e.g., `damageAmount`, `targetPosition`).
- **Constants & Static Readonly**: `PascalCase` (e.g., `MaxPlayersLimit`).

## 2. Unity-Specific Serialization & Encapsulation
- **Inspector Variables**: NEVER declare a field `public` purely to expose it to the Unity Inspector. Modifying state directly breaks encapsulation.
  - *Correct*: `[SerializeField] private float _moveSpeed;`
  - *Incorrect*: `public float MoveSpeed;`
- **Attribute Styling**: For multiple meta instances, stack them vertically or cleanly chunk them.
```csharp
[Header("Locomotion")]
[Range(0f, 10f)]
[SerializeField] private float _speed = 5f;
```
- **Component Dependencies**: If a script inherently manipulates a specific component (e.g., `Rigidbody2D`), it must establish safety by declaring `[RequireComponent(typeof(Rigidbody2D))]` at the top of the class.

## 3. Formatting & Bracing
- **Allman Style**: Opening and closing curly braces `{ }` MUST be placed on new lines extending parallel blocks for namespaces, classes, methods, and control flow wrappers (`if`, `for`, `while`).
- **Readability via Guard Clauses**: Instead of nesting deeply (`if (true) { if (true) { ... } }`), invert the flow and `return early`. Code should be vertical, not slanting to the right side of the screen.

## 4. Documentation & Comments

### Language
- **All code, comments, and documentation MUST be written in English.** No exceptions — including `[Header]` labels, `[Tooltip]` strings, log messages, and inline remarks.

### XML Documentation (Required)
Every `public` or `internal` type and member **must** have an XML doc comment:

```csharp
/// <summary>
/// Manages the player's health state and exposes damage/heal operations.
/// Fires <see cref="OnDied"/> when health reaches zero.
/// </summary>
public sealed class HealthSystem : IHealthSystem
{
    /// <summary>Fires when health reaches zero. Guaranteed to fire exactly once per life.</summary>
    public event Action OnDied;

    /// <summary>
    /// Applies damage to the current health value.
    /// Clamps result to zero and fires <see cref="OnDied"/> if lethal.
    /// </summary>
    /// <param name="amount">Positive damage value. Negative values are ignored.</param>
    public void TakeDamage(float amount) { ... }

    /// <summary>Returns health as a normalized value between 0 and 1.</summary>
    public float GetNormalizedHealth() { ... }
}
```

### Inline Comments
- Use inline comments (`//`) **only** to explain *why*, never *what* — the code itself explains what.
  - ✅ `// Delay one frame to allow physics to settle after teleport`
  - ❌ `// Subtract damage from health` (obvious from code)
- Never leave commented-out code blocks in committed files. Use `git stash` or a feature branch instead.

### Inspector Labels (English-only)
```csharp
[Header("Movement Settings")]          // ✅ English
[Tooltip("Base movement speed in m/s. Affected by terrain multiplier.")]
[SerializeField] private float _speed;

// ❌ Never:
[Header("Cài đặt di chuyển")]
```
