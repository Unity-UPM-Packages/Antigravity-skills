# Workflow: Addressables & Asset Pipeline

## Objective
When tasked with spawning assets dynamically, managing large environments, or engineering loading screens, enforce the standard of memory safety via Unity Addressables. 

## Execution Sequence
1. **Constraint Check**: Explicitly forbid the use of `Resources.Load()` or placing heavy items inside the `/Resources` folder unless rigidly mandated by legacy constraints.
2. **Addressable Invocation**: 
   - Load assets safely utilizing `Addressables.InstantiateAsync()` or `Addressables.LoadAssetAsync<T>()`.
   - Explicitly handle error-checking on the `AsyncOperationHandle` bindings (Event completion or `await`).
3. **Memory Release Enforcement**:
   - For every asset loaded dynamically, you MUST provide the architectural logic to release it later (`Addressables.Release()` or `Addressables.ReleaseInstance()`). A loading script lacking a release trajectory is an architectural violation.
4. **Disposal Integration**: Cleanly wire memory release hooks into Unity's component lifecycle (`OnDestroy`) or via custom disposal interfaces governed by the `modular-design.md` rule.
