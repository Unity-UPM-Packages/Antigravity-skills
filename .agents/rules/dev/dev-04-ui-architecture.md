---
trigger: model_decision
glob:
description: Enforces MVP pattern and strict separation between game logic and UI views.
---

# Rule 04: UI Architecture (MVP Pattern)

## Overview
UI code often becomes the dirtiest part of Unity projects. To maintain robustness, Game Logic and User Interface elements must be radically separated. Views are passive display surfaces — they never own logic, never query game state, and never make decisions.

## Directives

### Zero Game Logic in Views
A script attached to a Button, Canvas, or any visual element (`View` layer) must NOT contain:
- Health calculations or game state logic
- Database queries or save/load calls
- Business rules or win/loss conditions

Views receive data via `Construct()` injection and display it. That is their only job.

### Data Binding via Events
- Subscribe to system events in `OnEnable` / via `Construct()`, unsubscribe in `OnDisable` / `OnDestroy`
- Use C# `Action` / `event` to push data changes from systems to views
- **Never** place UI text updates inside `Update()` — update only when data changes via events
- Prefer plain `Action`/`event` over reactive frameworks unless the project already uses R3

### Presenter / Controller Intermediary
If a view needs to react to multiple systems simultaneously, introduce a `Presenter` class:
```csharp
// HUDPresenter.cs — mediates between systems and HUD view
public sealed class HUDPresenter
{
    private readonly IHealthSystem _health;
    private readonly IScoreSystem _score;
    private readonly HUDView _view;

    public HUDPresenter(IHealthSystem health, IScoreSystem score, HUDView view)
    {
        _health = health;
        _score = score;
        _view = view;

        _health.OnHealthChanged += _view.UpdateHealthBar;
        _score.OnScoreChanged += _view.UpdateScore;
    }
}
```

### Reactive Framework (Optional)
If the project already integrates **R3** (successor to UniRx), use Observable chains for complex multi-source data binding. Do not introduce R3 solely for simple one-to-one event bindings — plain `Action` is sufficient and has zero overhead.
