# Rule 00: C# & Unity Coding Conventions

## Overview
A strict standardization for naming conventions, code formatting, and structure syntax to maintain absolute readability across the entire Unity project. Adherence to this file guarantees a uniform codebase regardless of which developer or AI generates the code.

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
