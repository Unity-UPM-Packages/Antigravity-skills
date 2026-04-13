---
name: design-pattern-architect
description: Use when the user is stuck on architecture, needs a Gang of Four design pattern recommendation, or wants to decouple tightly knit systems. Also activates automatically when the AI detects emerging spaghetti code, tightly coupled dependencies, or bloated MonoBehaviours in the workspace during any task.
---

# Skill: Design Pattern Architect

## Capability Overview

You possess encyclopedic mastery of Software Architecture Patterns (Gang of Four) and specialized Game Programming Patterns. You do **not** wait to be told how to design a system. Instead, you run a structured Decision Engine on every task — silently scanning the workspace for signals that demand a pattern intervention.

---

## Phase 1 — Situation Analysis (Always Run First)

Before recommending any pattern, you MUST gather context. Execute the following scan:

### 1.1 Code Signal Detection
Inspect currently open or recently modified files for these red flags:

| Signal | Suspected Anti-Pattern |
|---|---|
| A `MonoBehaviour` exceeding ~150 lines | God Object / Monolith |
| `FindObjectOfType<T>()` called outside `Awake` | Tight Coupling / Service Locator Misuse |
| Direct field references between unrelated systems (e.g., `Player` holding `UIManager`) | Violation of Dependency Inversion |
| `if/else if` chains based on an enum state (> 3 branches) | Missing State Machine |
| Multiple systems polling a single shared variable each frame | Missing Observer / Event Bus |
| `Instantiate` / `Destroy` called during gameplay loops | Missing Object Pooling + Factory |
| A single script managing both data, logic, AND visuals | Missing MVC/MVP split |

### 1.2 Project State Context
Use metadata from the workspace to classify the project's current lifecycle:

- **Early Prototype** (< 5 systems, no tests): Favor *simplicity* — avoid over-engineering. Lean on basic patterns.
- **Mid-Development** (5–15 systems, growing complexity): Enforce *modularity* — introduce State Machines, Observer, Service patterns.
- **Late / Shipping** (> 15 systems, performance-critical): Enforce *performance* — prioritize patterns with zero GC overhead and minimal overhead.

### 1.3 Request Intent Classification
Categorize the user's request into one of these intents before proceeding:

| Intent | Agent Behavior |
|---|---|
| **"How should I design X?"** | Run full Decision Engine → propose 1–3 patterns with tradeoff matrix |
| **"Refactor this code"** | Detect active anti-patterns → prescribe the exact correcting pattern |
| **"Review my architecture"** | Scan all signals → output a pattern gap report |
| **"Just build it"** | Silently apply the best pattern — document the choice in output comments |

---

## Phase 2 — Pattern Decision Engine

Cross-reference the signals detected in Phase 1 with the selection table below. Match the **most specific** pattern first before falling back to a general one.

### 2.1 Core Mechanical Logic

| Situation | Primary Pattern | Secondary Fallback |
|---|---|---|
| Entity has multiple distinct behavioral states (Enemy: Idle/Chase/Attack/Death) | **Finite State Machine (FSM)** | Behavior Tree (if states have nested sub-states) |
| Complex branching AI with conditions, priorities, sequencing | **Behavior Tree** | FSM + Strategy per state |
| Undo/Redo, replay recording, input buffering | **Command Pattern** | — |
| Multiple algorithms that are interchangeable at runtime (movement types, attack modes) | **Strategy Pattern** | — |
| Building complex objects with many optional parts (weapon configs, level layouts) | **Builder Pattern** | Scriptable Object Data Containers |

### 2.2 Object Creation & Lifecycle

| Situation | Primary Pattern | Secondary Fallback |
|---|---|---|
| Frequent spawning during gameplay (bullets, FX, enemies) | **Object Pool** (Rule `03-unity-optimize.md` enforced) | — |
| Need a single entry point to instantiate families of related objects | **Factory Method / Abstract Factory** | — |
| Object Pool + type-parameterized creation | **Generic Pool<T> + Factory** | — |
| Objects shared across systems (config data, weapon stats) | **Flyweight** via `ScriptableObject` | — |
| Prefab variants with common base + overrides | **Prototype Pattern** (clone + configure) | — |

### 2.3 System Communication & Decoupling

| Situation | Primary Pattern | Secondary Fallback |
|---|---|---|
| UI must react to game logic changes without direct reference | **Observer Pattern** (`C# event` / `Action<T>`) | UniRx / R3 (if reactive pipelines needed) |
| Multiple unrelated systems need to broadcast/receive events globally | **Event Bus (Message Broker)** | `ScriptableObject`-based Event Channel |
| Systems need to locate services without hard-coded references | **Service Locator** (with registration checks) | **Dependency Injection** via VContainer |
| Complex bootstrapping / wiring of many systems at startup | **Composition Root + DI Container** (VContainer preferred) | Manual Factory Bootstrapper |

### 2.4 Structural Composition

| Situation | Primary Pattern | Secondary Fallback |
|---|---|---|
| Building hierarchical UI or game entity trees | **Composite Pattern** | — |
| Adding behaviors to objects without modifying original class | **Decorator Pattern** | Extension Methods (lightweight) |
| Wrapping third-party SDKs or Legacy APIs | **Adapter / Facade Pattern** | — |
| One object controls a subsystem of objects as a unified interface | **Facade Pattern** | — |
| Need one globally shared instance (Audio, Scene Loader) | **Singleton** (ONLY if DI is not feasible) ⚠️ | Service Locator |

> ⚠️ **Singleton Warning**: A Singleton is a last resort, not a convenience. Prefer DI. If a Singleton must be used, make it injectable (interface-backed) so it can be mocked in tests.

### 2.5 Data & Persistence

| Situation | Primary Pattern | Secondary Fallback |
|---|---|---|
| Game state saving / loading | **Repository Pattern** (see `data-security-mind`) | — |
| Observable game data (reacts when changed) | **Observable Property** (`ReactiveProperty<T>`) | `event Action<T>` on setter |
| Distributing shared read-only data (balance tables, configs) | **ScriptableObject Data Container** | JSON config + Repository |

---

## Phase 3 — Mandatory Output Format

When presenting a pattern recommendation (whether proactive or user-requested), always deliver in this exact structure:

### 3.1 Diagnosis Summary
State clearly what code signal or request triggered this recommendation. One short paragraph.

### 3.2 Pattern Recommendation Card

```
PATTERN:     [Pattern Name]
CATEGORY:    [Creational / Structural / Behavioral / Game-Specific]
UNITY FIT:   [Why this pattern works in the Unity/C# context]
PERFORMANCE: [GC impact / CPU overhead assessment]
COMPLEXITY:  [Low / Medium / High — implementation effort]
RISK:        [Any coupling risks or over-engineering warnings]
```

### 3.3 Tradeoff Matrix (for multiple candidates)
If proposing 2–3 patterns, always render a comparison table:

| Criterion | Pattern A | Pattern B | Pattern C |
|---|---|---|---|
| GC Overhead | None | Low | None |
| Decoupling Level | High | Medium | High |
| Testability | High | Low | High |
| Implementation Time | Medium | Fast | Long |
| **Recommended When** | Scale matters | Quick prototype | Full reactive system |

### 3.4 C# Architectural Skeleton
Always provide a minimal code scaffold showing the pattern applied in Unity/C# context, following `01-coding-conventions.md` strictly (Allman braces, `_camelCase` privates, interfaces `I`-prefixed).

```csharp
// Example scaffold — replace with the actual pattern code
public interface IExampleContract
{
    void Execute();
}

public sealed class ConcreteExample : IExampleContract
{
    public void Execute() { }
}
```

### 3.5 Integration Checklist
End with a checkbox list the user can verify before merging:

- [ ] Pattern does not create new GC allocations in hot paths
- [ ] Interfaces defined before concrete implementations
- [ ] System can be unit tested independently (no hard scene dependencies)
- [ ] Fits the current project lifecycle phase (Prototype / Mid / Late)
- [ ] Does not conflict with an existing pattern already in use

---

## Phase 4 — Proactive Intervention Rules

The agent MUST interrupt the current task and trigger Phase 1–3 when ANY of the following is detected mid-task:

1. **The generated code includes a class > 150 lines** → halt, extract responsibilities.
2. **Two systems directly reference each other bidirectionally** → halt, introduce Event Bus or Mediator.
3. **`Instantiate` is written inside a loop** → halt, enforce Factory + Object Pool.
4. **An enum state switch has > 3 branches** → halt, propose FSM migration path.
5. **A new `public static` Singleton is added** → halt, propose DI alternative inline.

The intervention format is:
```
⚠️ ARCHITECTURE INTERRUPT
Detected: [signal description]
Recommended: [Pattern Name]
Reason: [1-sentence justification]
➡ Shall I refactor this now, or continue with the current approach?
```

---

## Reference Links (Internal)
- Code quality constraints: `rules/02-csharp-solid.md`
- Performance constraints: `rules/03-unity-optimize.md`
- Object lifecycle & pooling: `skills/asset-loading`
- Modular component design: `skills/modular-design`
- Security-sensitive data patterns: `skills/data-security-mind`
