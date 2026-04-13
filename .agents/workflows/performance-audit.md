# Workflow: Performance Audit

A comprehensive performance sweep across 5 tracks: CPU (GC), Memory, GPU/Rendering, UI, and Asset Pipeline. Output is a prioritized report with issues ranked by severity.

---

## Step 1 — Scan for GC Allocations in Hot Paths

Search for forbidden allocation patterns inside `Update`, `FixedUpdate`, `LateUpdate`:

```bash
grep -rn "new " Assets/Scripts --include="*.cs" | grep -v "//"
```

For each match, classify:
- `new List<>()` inside Update → **CRITICAL**: cache in `Awake`
- `new WaitForSeconds()` inside a Coroutine loop → **HIGH**: cache as a field
- String interpolation `$"..."` or `+` concatenation → **HIGH**: use `StringBuilder` or composite format
- LINQ (`Where`, `Select`, `FirstOrDefault`) → **MEDIUM**: replace with a `for` loop

---

## Step 2 — Scan for Anti-Patterns

```bash
# Expensive scene lookups
grep -rn "FindObjectOfType\|FindObjectsOfType" Assets/Scripts --include="*.cs"

# Runtime Instantiate/Destroy (must use Object Pooling)
grep -rn "Instantiate\|Destroy(" Assets/Scripts --include="*.cs"

# Direct asset loading (must use Addressables)
grep -rn "Resources\.Load" Assets/Scripts --include="*.cs"

# Camera.main in Update (expensive lookup every call)
grep -rn "Camera\.main" Assets/Scripts --include="*.cs"
```

Every hit is marked **CRITICAL** — these are ship blockers.

---

## Step 3 — Audit Canvas / UI Structure

- [ ] Canvas correctly partitioned (Background / Main / HUD / Overlay)?
- [ ] HUD canvas (health, ammo, timer) isolated on its own Canvas?
- [ ] All decorative `Image` components have `raycastTarget = false`?
- [ ] Any nested `LayoutGroup` deeper than 2 levels? (triggers cascade rebuild)
- [ ] Dynamic text uses `TextMeshProUGUI`, not legacy `Text`?
- [ ] `ScrollView` with > 10 items uses a pool?

---

## Step 4 — Audit Asset Pipeline

- [ ] All textures compressed with a mobile-appropriate format (ASTC)?
- [ ] No UI texture larger than 2048×2048?
- [ ] Audio clips > 200 KB have `Load In Background = true`?
- [ ] Audio clips use `Vorbis` compression on mobile?
- [ ] All prefabs loaded via Addressables (no `Resources.Load`)?
- [ ] No unused scenes present in Build Settings?

---

## Step 5 — Detect Memory Leaks

Find event subscriptions without a matching unsubscribe:

```bash
grep -rn "+=" Assets/Scripts --include="*.cs" | grep -v "//"
```

For each `event +=`, verify the owning class has a corresponding `-=` inside `OnDestroy` or `OnDisable`.

Also check unbounded collections used as pools — ensure they have a max-size cap.

---

## Step 6 — Generate Audit Report

Summarize results in this format:

```
# Performance Audit Report — [YYYY-MM-DD]

## CRITICAL (ship blockers — fix immediately)
- [ ] [File:Line] Description → Recommended fix

## HIGH (fix this sprint)
- [ ] [File:Line] Description → Recommended fix

## MEDIUM (backlog)
- [ ] [File:Line] Description → Recommended fix

## PASSING ✅
- Zero GC confirmed in: [list of systems checked]
- Canvas partitioned correctly
- [other passing items]
```

---

## Step 7 — Fix CRITICAL Issues

For each CRITICAL issue:
1. Apply fix following `skills/unity-profiler-mind`
2. Re-run the corresponding grep to confirm the fix
3. Commit: `perf(<scope>): <description of fix>`

---

## Cross-Skill References
- Deep investigation protocol → `skills/unity-profiler-mind`
- Canvas optimization patterns → `skills/ui-performance`
- Asset loading best practices → `skills/asset-loading`
