# Rule 03: UI Architecture (MVC/MVP approach)

## Overview
UI code often becomes the dirtiest part of Unity projects. To maintain robustness, Game Logic and User Interface elements must be radically separated.

## Directives
- **Zero Game Logic in UI Views**:
  - A script attached to a Button or Canvas element (`View` behavior) cannot contain health calculations, database queries, or core game logic.
- **Data Binding & Connectors**:
  - Use `Presenter` or `Controller` intermediary classes to link pure data models with the visual elements.
- **Event-Driven**:
  - Utilize C# `Action`, `event`, or a reactive framework (e.g., `UniRx`) to update the DOM elements only when data changes. DO NOT place UI text updates inside generic `Update()` loops.
