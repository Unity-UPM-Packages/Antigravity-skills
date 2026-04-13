---
name: code-review
description: Use when asked to review, audit, or inspect C# code in a Unity project. Also activates proactively when the AI reads any code and detects performance issues, architectural violations, missing null checks, GC allocations in hot paths, or security concerns — even if the user did not explicitly request a review.
---

# Workflow: Code Review & Optimization Audit

## Objective
Critique existing code and identify architectural and performance anti-patterns. Output is always a **structured remediation report with exact code fixes** — never vague advice like "consider refactoring this."

---

## Audit Sequence (Run All Five)

### Audit 1 — SOLID & Architecture
Cross-reference against `rules/02-csharp-solid.md`:

| Check | Pass Condition | Fail → Action |
|---|---|---|
| Single Responsibility | Class < 150 lines, one reason to change | Extract class / service |
| Open/Closed | New behavior via extension, not modification | Introduce Strategy or Decorator |
| Liskov Substitution | Subtypes honor base contracts fully | Remove or fix inheritance |
| Interface Segregation | Interfaces < 5 methods, no forced unused impl | Split into focused interfaces |
| Dependency Inversion | Depends on `IAbstraction`, not `ConcreteClass` | Inject interface, remove `new` |
| God Object | No class manages > 2 unrelated systems | Split into focused services |
| Tight Coupling | No direct concrete cross-system field references | Event Bus or DI injection |
| Script Length | No MonoBehaviour > 200 lines | Extract Pure Logic to plain C# class |

### Audit 2 — Unity Performance (Zero GC)
Cross-reference against `rules/03-unity-optimize.md`:

| Anti-Pattern | Detection Signal | Required Fix |
|---|---|---|
| Hot-path allocation | `new` / LINQ / string concat in `Update` | Pre-allocate, cache, use `StringBuilder` |
| Late GetComponent | `GetComponent<T>()` outside `Awake/Start` | Cache in `Awake` as `_field` |
| FindObjectOfType misuse | Called outside init phase | Inject via DI or cache at startup |
| Gameplay Instantiate | `Instantiate` inside gameplay loop | Object Pool (Rule 03 enforced) |
| String tag comparison | `tag == "Enemy"` | `CompareTag("Enemy")` or Enum map |
| Missing null guard | Optional dependency with no null check | Guard clause + early return |
| Uncached Animator hash | `animator.SetTrigger("Jump")` in loop | `Animator.StringToHash` cached |

### Audit 3 — UI Logic Bleed
Cross-reference against `rules/04-ui-architecture.md`:
- UI scripts must NEVER contain game logic (damage calc, weapon switching, AI decisions)
- Presenters/ViewModels must isolate Model data from View components
- UI must react to events — it must NEVER poll state each frame via `Update`
- `Canvas.ForceUpdateCanvases()` called manually is always a code smell

### Audit 4 — Async & Coroutine Safety
Cross-reference against `rules/07-async-coroutines.md`:

| Check | Required Behavior |
|---|---|
| Coroutine lifetime | Must be stopped via `StopCoroutine` before `OnDestroy` |
| `async` Task cancellation | Must use `CancellationTokenSource` destroyed in `OnDestroy` |
| `await` error handling | Must be wrapped in `try/catch` with logged exceptions |
| `async void` usage | Forbidden except for Unity event handlers — use `async Task` |
| UnityWebRequest | Must be disposed in `finally` block |

### Audit 5 — Data Integrity & Security
- Sensitive values (currency, XP, stats) stored in `PlayerPrefs` → **Critical violation** — flag for `data-security-mind` review
- Save data written to disk without checksum/signature → **Warning** — flag for hash enforcement
- `public static` mutable state accessible globally → **Warning** — encapsulate, consider DI
- API keys or auth tokens hardcoded in source → **Critical** — must use `Resources` encrypted config or server-side

---

## Output Format (Mandatory)

Always structure the output exactly as follows — no exceptions:

```markdown
## Code Review Report: [ClassName.cs]

### 🔴 Critical Issues (Block Merge)
| # | Line | Issue | Exact Fix |
|---|------|-------|-----------|
| 1 | L.42 | `new List<>()` allocated in Update() | Pre-allocate in Awake, reuse |

### 🟡 Warnings (Fix This Sprint)
| # | Line | Issue | Recommendation |

### 🟢 Suggestions (Backlog)
| # | Line | Observation | Improvement |

### ✅ Refactored Code
[Provide the exact corrected code for EVERY Critical issue — no partial fixes]
```

---

## Reviewer Decision Rules
- **> 3 Critical issues in one file** → Recommend full rewrite with `create-feature` SOP.
- **Architecture violation detected** → Cross-reference `design-pattern-architect` for the structural fix.
- **Performance violation detected** → Cross-reference `unity-profiler-mind` for profiling guidance.
- **Never** provide generic advice. Every finding must include the exact replacement code.
- **Always** check if a fix in one place requires corresponding changes elsewhere (cascading refactor awareness).

---

## Cross-Skill References
- Architecture decisions for refactoring → `skills/design-pattern-architect`
- Performance deep-dive → `skills/unity-profiler-mind`
- Data/save security → `skills/data-security-mind`
- Writing replacement feature from scratch → `skills/create-feature`
