---
name: code-review
description: Use when asked to review C# code, check for performance leaks, or audit workflows in a Unity project.
---

# Workflow: Code Review & Optimization Audit

## Objective
Critique existing code and identify architectural and performance anti-patterns.

## Execution Sequence
1. **SOLID Audit**: Review against `01-csharp-solid.md`. Flag tightly coupled code, classes over 100 lines, missing interfaces, or bloat.
2. **Performance Audit**: Review against `02-unity-optimize.md`. Flag object creations (`new`), allocations, LINQ extensions in loops, or dynamic `GetComponent()` calls inside `Update`.
3. **UI Logic Bleed Audit**: Review against `03-ui-architecture.md`. Ensure presenters isolate models from view components effectively.
4. **Actionable Output**: Output a structured remediation list providing the exact refactored code fixes rather than generalized advice.

