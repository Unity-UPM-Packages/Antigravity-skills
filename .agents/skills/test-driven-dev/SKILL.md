---
name: test-driven-dev
description: Use when implementing logic to ensure NUnit PlayMode and EditMode tests are drafted and verify successful code passes.
---

# Workflow: Test-Driven AI (TDD)

## Objective
Enforce autonomous verification of mechanics. The AI must aim to prove its code functions correctly via the Unity Test Framework before requesting manual human review, shifting the user's role to a pure "Reviewer".

## Execution Sequence
1. **Mathematical Validation First**: Before writing production C# logic, draft the state/math validation tests utilizing `NUnit` assertions.
2. **Implementation**: Write the logic explicitly resolving the failing tests.
3. **Mode Distinctiveness**: Explicitly categorize tests into `/EditMode` (for rapid logic/math verification devoid of GameObjects) and `/PlayMode` (for `MonoBehaviour` Physics/Collision/Time simulation).
4. **Self-Correction**: Anticipate test failures. Iteratively refactor the codebase to pass all unit criteria before delivering the final PR summary to the user.

