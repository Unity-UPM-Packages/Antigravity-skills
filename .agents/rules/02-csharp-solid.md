# Rule 02: C# SOLID & Script Composition

## Overview
This rule enforces strict architectural design to maintain codebase sanity and scalability. Avoid "Spaghetti Logic" and bloated monolithic `MonoBehaviour` classes.

## Directives
- **Short & Concise**: Scripts should be minimal and focused. If a script exceeds generic boundaries (e.g., > 100-200 lines), it usually violates the Single Responsibility Principle.
- **Script Composition**: Rely heavily on Component Pattern / Composition over Inheritance. Combine tiny components to create complex behaviors.
- **MonoBehaviour Usage**: 
  - Restrict `MonoBehaviour` mainly to the "View" layer (visuals, Unity lifecycles, Collisions).
  - Pure Logic, Data Models, and Services MUST be plain C# classes (`class` or `struct`) unaffected by engine lifecycles.
- **Interfaces (Contracts)**: Always define boundaries utilizing C# `interface`. This enforces loose coupling and simplifies unit testing later on.
- **SOLID Validation**: Always audit your design against SOLID (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion) before implementing.
