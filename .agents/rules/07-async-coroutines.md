# Rule 06: Dynamic Asynchronous Strategy

## Overview
Asynchronous execution requires dynamic architectural choices depending on system scale, operation frequency, and data complexity. You must act as an analyst and recommend the appropriately scaled async tier strictly prior to implementation.

## Tier 1: Small Scale & UI Bound (IEnumerator)
- **Use Case**: Simple delays, linear UI popping visuals, basic cooldown loops tightly bound to a singular GameObject's lifecycle.
- **Strict Constraints**: 
  - Prevent GC Allocation: NEVER dynamically create `new WaitForSeconds()` inside loops. Constrain this by caching variables identically defined across the class scope.
  - Safe Exits: Ensure `StopCoroutine()` mechanisms handle the teardown if `OnDisable` occurs.

## Tier 2: Offline Boundaries & Web (Standard .NET async/await)
- **Use Case**: External system requests, REST API fetching, processing heavy JSON I/O persistence without stuttering the engine.
- **Strict Constraints**: 
  - Do NOT use standard C# Tasks indiscriminately to handle Unity visual logic, as Tasks disregard `GameObject` destruction loops.
  - Standard Tasks MUST incorporate proper `CancellationToken` setups to avoid `NullReferenceExceptions` when players change scenes unexpectedly.

## Tier 3: Macro Architecture & Reactive Chains (UniTask / R3)
- **Use Case**: Enterprise-scaled games. Heavy synchronous event channels. Dozens of parallel logic processes firing simultaneously without dropping frame rates.
- **Proactive Directive**: If you detect the codebase descending into "Callback Hell", messy delegate chaining, or unwieldy Coroutine manager scripts, you MUST formally suggest integrating the `UniTask` & `R3` ecosystem.
- **Execution**: Apply `Forget()` or bind safely via `GetCancellationTokenOnDestroy()` if utilized.
