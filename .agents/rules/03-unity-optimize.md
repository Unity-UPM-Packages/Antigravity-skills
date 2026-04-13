# Rule 03: Unity Mobile Optimization

## Overview
Mobile games are extremely constrained by CPU, RAM, and thermal limitations. The engine's Garbage Collector (GC) causes stuttering/lag spikes if memory is poorly managed. Strict adherence to Zero Allocation is mandated.

## Directives
- **Zero GC in Hot Paths**:
  - Never allocate dynamically inside `Update()`, `FixedUpdate()`, or `LateUpdate()`.
  - Avoid creating `new` objects, using string concatenation (`"A" + "B"`), or generating Closures via `LINQ` in loop sequences.
- **Explicit Caching**: 
  - Preemptively cache references to components (`GetComponent<T>`), Physics casts, and complex calculations during the `Awake()` or `Start()` phases.
- **String Hashing**: 
  - Never use strings for comparisons in continuous logic. 
  - Map Tags to an Enum, and ALWAYS use `CompareTag()`. 
  - For Animations, use `Animator.StringToHash()`.
- **Object Pooling**:
  - `Instantiate` and `Destroy` calls during gameplay are strictly forbidden. You must fallback to using an Object Pooling architecture for bullets, items, UI elements, etc.
