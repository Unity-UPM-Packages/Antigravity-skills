---
name: dev-input-system
description: Use when architecting player input handling using Unity's new Input System, designing action maps, or implementing mobile touch patterns.
---

# Skill: Input System Architecture

---

## Core Principle: Input Abstraction Layer

No gameplay system should query `InputAction` directly. All input must flow through an `IInputProvider` interface. This allows:
- Easy swapping between keyboard, gamepad, and touch
- Mocking in unit tests without simulating hardware
- Input recording and replay

```csharp
// IInputProvider.cs — in Game.Core.asmdef
public interface IInputProvider
{
    // Movement
    Vector2 MovementDirection { get; }
    bool IsJumpPressed { get; }
    bool IsJumpHeld { get; }

    // Combat
    bool IsAttackPressed { get; }
    bool IsAimHeld { get; }
    Vector2 AimDirection { get; }

    // UI / Meta
    bool IsPausePressed { get; }

    // Events (for one-shot actions — preferred over polling)
    event Action OnJumpPressed;
    event Action OnAttackPressed;
    event Action OnPausePressed;
}
```

---

## Input Action Map Design

In the Unity Input System Asset, organize Action Maps by context:

```
PlayerInputActions.inputactions
├── ActionMap: Gameplay
│   ├── Move          (Value, Vector2)    — Stick / WASD / Touchpad
│   ├── Jump          (Button)            — Space / A Button / Touch region
│   ├── Attack        (Button)            — Left Click / X Button / Touch region
│   ├── Aim           (Value, Vector2)    — Mouse Delta / Right Stick / Gyro
│   └── Pause         (Button)            — Escape / Start / Back button
└── ActionMap: UI
    ├── Navigate      (Value, Vector2)
    ├── Submit        (Button)
    └── Cancel        (Button)
```

Enable/disable Action Maps via code when context changes (gameplay → menu → cutscene).

---

## InputProvider Implementation

```csharp
// InputProvider.cs — in Game.Gameplay.asmdef (wraps generated C# class)
public sealed class InputProvider : IInputProvider, IDisposable
{
    private readonly PlayerInputActions _actions;

    // Cached values — updated via callbacks (no per-frame polling overhead)
    private Vector2 _movementDirection;
    private Vector2 _aimDirection;
    private bool _isJumpHeld;
    private bool _isAimHeld;

    public Vector2 MovementDirection => _movementDirection;
    public Vector2 AimDirection => _aimDirection;
    public bool IsJumpHeld => _isJumpHeld;
    public bool IsAimHeld => _isAimHeld;
    public bool IsJumpPressed { get; private set; }
    public bool IsAttackPressed { get; private set; }
    public bool IsPausePressed { get; private set; }

    public event Action OnJumpPressed;
    public event Action OnAttackPressed;
    public event Action OnPausePressed;

    public InputProvider()
    {
        _actions = new PlayerInputActions();
        _actions.Gameplay.Enable();

        // Subscribe to input events — no per-frame polling
        _actions.Gameplay.Move.performed += ctx => _movementDirection = ctx.ReadValue<Vector2>();
        _actions.Gameplay.Move.canceled += _ => _movementDirection = Vector2.zero;

        _actions.Gameplay.Jump.performed += _ =>
        {
            IsJumpPressed = true;
            _isJumpHeld = true;
            OnJumpPressed?.Invoke();
        };
        _actions.Gameplay.Jump.canceled += _ =>
        {
            _isJumpHeld = false;
            IsJumpPressed = false;
        };

        _actions.Gameplay.Attack.performed += _ =>
        {
            IsAttackPressed = true;
            OnAttackPressed?.Invoke();
        };
        _actions.Gameplay.Attack.canceled += _ => IsAttackPressed = false;

        _actions.Gameplay.Pause.performed += _ =>
        {
            IsPausePressed = true;
            OnPausePressed?.Invoke();
        };
    }

    public void SwitchToUIMap()
    {
        _actions.Gameplay.Disable();
        _actions.UI.Enable();
    }

    public void SwitchToGameplayMap()
    {
        _actions.UI.Disable();
        _actions.Gameplay.Enable();
    }

    public void Dispose()
    {
        _actions.Gameplay.Disable();
        _actions.UI.Disable();
        _actions.Dispose();
    }
}
```

---

## Mobile Touch Patterns

### Virtual Joystick (Movement)
Use a dedicated Virtual Joystick component that reads `Touch` input and maps to `IInputProvider.MovementDirection`. Do **not** update `MovementDirection` directly from the joystick — the joystick fires a `Vector2` event that the `InputProvider` consumes.

```csharp
public sealed class VirtualJoystick : MonoBehaviour, IPointerDownHandler, IDragHandler, IPointerUpHandler
{
    [SerializeField] private RectTransform _knob;
    [SerializeField] private float _maxRadius = 80f;

    public event Action<Vector2> OnDirectionChanged;

    public void OnDrag(PointerEventData eventData)
    {
        Vector2 delta = eventData.position - (Vector2)transform.position;
        Vector2 direction = Vector2.ClampMagnitude(delta, _maxRadius) / _maxRadius;
        _knob.anchoredPosition = direction * _maxRadius;
        OnDirectionChanged?.Invoke(direction);
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        _knob.anchoredPosition = Vector2.zero;
        OnDirectionChanged?.Invoke(Vector2.zero);
    }

    public void OnPointerDown(PointerEventData eventData) { }
}
```

### Touch Action Buttons (Attack, Jump)
Use `IPointerDownHandler` / `IPointerUpHandler` on UI Buttons — map directly to the `IInputProvider` events via `Construct()`:

```csharp
public sealed class TouchAttackButton : MonoBehaviour, IPointerDownHandler, IPointerUpHandler
{
    private IMobileInputBridge _bridge;

    /// <summary>Injects the input bridge. Call from the scene Bootstrapper.</summary>
    public void Construct(IMobileInputBridge bridge) => _bridge = bridge;

    public void OnPointerDown(PointerEventData _) => _bridge.NotifyAttackPressed();
    public void OnPointerUp(PointerEventData _) => _bridge.NotifyAttackReleased();
}
```

### Screen Swipe / Gesture Detection
For gesture-based input (swipe-to-dodge, pinch-to-zoom).
**Must use `EnhancedTouch` from the new Input System — never `Input.GetTouch()` (legacy).**

```csharp
using UnityEngine.InputSystem.EnhancedTouch;
using Touch = UnityEngine.InputSystem.EnhancedTouch.Touch;

/// <summary>
/// Detects linear swipe gestures using the new Input System's EnhancedTouch API.
/// Fires <see cref="OnSwipeDetected"/> with a normalized direction vector.
/// </summary>
public sealed class SwipeDetector : MonoBehaviour
{
    [SerializeField] private float _minSwipeDistance = 50f;

    /// <summary>Fires when a swipe gesture exceeds the minimum distance threshold.</summary>
    public event Action<Vector2> OnSwipeDetected;

    private Vector2 _startPos;

    private void OnEnable()
    {
        EnhancedTouchSupport.Enable();
        Touch.onFingerDown += HandleFingerDown;
        Touch.onFingerUp += HandleFingerUp;
    }

    private void OnDisable()
    {
        Touch.onFingerDown -= HandleFingerDown;
        Touch.onFingerUp -= HandleFingerUp;
    }

    private void HandleFingerDown(Finger finger) => _startPos = finger.screenPosition;

    private void HandleFingerUp(Finger finger)
    {
        Vector2 delta = finger.screenPosition - _startPos;
        if (delta.magnitude >= _minSwipeDistance)
            OnSwipeDetected?.Invoke(delta.normalized);
    }
}
```

---

## Input Context Switching
When transitioning between game states, always switch the active control scheme:

| Game State | Active Map | Action |
|---|---|---|
| Gameplay | `Gameplay` | `_input.SwitchToGameplayMap()` |
| Pause Menu | `UI` | `_input.SwitchToUIMap()` |
| Cutscene | None | `_actions.Disable()` |
| Dialogue | `UI` (navigate only) | `_input.SwitchToUIMap()` + disable gameplay |

---

## Testing Input Without Hardware

```csharp
// MockInputProvider.cs — for unit tests
public sealed class MockInputProvider : IInputProvider
{
    public Vector2 MovementDirection { get; set; }
    public bool IsJumpPressed { get; set; }
    public bool IsJumpHeld { get; set; }
    public bool IsAttackPressed { get; set; }
    public bool IsAimHeld { get; set; }
    public Vector2 AimDirection { get; set; }
    public bool IsPausePressed { get; set; }

    public event Action OnJumpPressed;
    public event Action OnAttackPressed;
    public event Action OnPausePressed;

    // Test helpers
    public void SimulateJump() => OnJumpPressed?.Invoke();
    public void SimulateAttack() => OnAttackPressed?.Invoke();
}
```

---

## Cross-Skill References
- Input context switching with game state → `skills/gameplay-loop-architect`
- Registering `InputProvider` in DI container → `skills/create-feature` (Step 5)
- Virtual joystick Canvas setup → `skills/ui-performance`
- Testing input-dependent systems → `skills/test-driven-dev` (MockInputProvider)
