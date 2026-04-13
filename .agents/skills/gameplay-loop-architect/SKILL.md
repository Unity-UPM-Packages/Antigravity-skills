---
name: gameplay-loop-architect
description: Use when designing the core game loop, managing game states (menu, playing, paused, game over), or architecting session lifecycle and win/lose conditions. Also activates proactively when the AI detects a GameManager singleton, state logic scattered across MonoBehaviours, or direct game-state checks like "if (GameManager.Instance.IsPlaying)" anywhere in the codebase.
---

# Skill: Gameplay Loop Architect

## Capability Overview
Governs the design of the game's top-level state machine — the system that orchestrates transitions between Menu, Loading, Playing, Paused, Victory, and Game Over states. This is the backbone of any game. Without a clean game loop architecture, every other system eventually couples to some variation of "if (GameManager.Instance.IsPlaying)" scattered across the codebase.

---

## The Game Loop State Machine

The game loop is implemented as a **Finite State Machine (FSM)** managed by a `GameStateMachine` service. Each state is a self-contained class that handles its own entry, logic, and exit behaviors.

```
AppEntry
    └── BootState
            ├── [Init complete] → MainMenuState
            │       └── [Play pressed] → LoadingState
            │               └── [Load complete] → GameplayState
            │                       ├── [Pause input] → PausedState
            │                       │       └── [Resume] → GameplayState
            │                       │       └── [Quit] → MainMenuState
            │                       ├── [Win condition] → VictoryState
            │                       │       └── [Continue] → MainMenuState (or next level)
            │                       └── [Lose condition] → GameOverState
            │                               └── [Retry] → LoadingState
            │                               └── [Menu] → MainMenuState
            └── [Fatal error] → ErrorState
```

---

## Interface Contracts

```csharp
// IGameState.cs — in Game.Core.asmdef
public interface IGameState
{
    void OnEnter();
    void OnExit();
    void OnUpdate(); // Called by state machine each frame — only if state needs tick
}

// IGameStateMachine.cs
public interface IGameStateMachine
{
    GameStateType CurrentState { get; }
    void Enter<TState>() where TState : IGameState;
    event Action<GameStateType> OnStateChanged;
}

public enum GameStateType
{
    Boot,
    MainMenu,
    Loading,
    Gameplay,
    Paused,
    Victory,
    GameOver,
}
```

---

## State Machine Implementation

```csharp
// GameStateMachine.cs — registered as singleton
public sealed class GameStateMachine : IGameStateMachine, ITickable
{
    private readonly Dictionary<Type, IGameState> _states;
    private IGameState _currentState;

    public GameStateType CurrentState { get; private set; }
    public event Action<GameStateType> OnStateChanged;

    public GameStateMachine(IEnumerable<IGameState> states)
    {
        _states = new Dictionary<Type, IGameState>();
        foreach (var state in states)
            _states[state.GetType()] = state;
    }

    public void Enter<TState>() where TState : IGameState
    {
        if (!_states.TryGetValue(typeof(TState), out IGameState nextState))
        {
            Debug.LogError($"[GameStateMachine] State not registered: {typeof(TState).Name}");
            return;
        }

        _currentState?.OnExit();
        _currentState = nextState;
        CurrentState = StateTypeFrom<TState>();
        _currentState.OnEnter();
        OnStateChanged?.Invoke(CurrentState);
    }

    public void Tick() => _currentState?.OnUpdate();

    private GameStateType StateTypeFrom<TState>() where TState : IGameState
        => typeof(TState).Name switch
        {
            nameof(BootState) => GameStateType.Boot,
            nameof(MainMenuState) => GameStateType.MainMenu,
            nameof(LoadingState) => GameStateType.Loading,
            nameof(GameplayState) => GameStateType.Gameplay,
            nameof(PausedState) => GameStateType.Paused,
            nameof(VictoryState) => GameStateType.Victory,
            nameof(GameOverState) => GameStateType.GameOver,
            _ => throw new ArgumentException($"Unknown state type: {typeof(TState).Name}")
        };
}
```

---

## Example State Implementations

### GameplayState
```csharp
public sealed class GameplayState : IGameState
{
    private readonly ISceneLoader _sceneLoader;
    private readonly IInputProvider _input;
    private readonly IWinConditionChecker _winChecker;
    private readonly IGameStateMachine _stateMachine;

    public GameplayState(
        ISceneLoader sceneLoader,
        IInputProvider input,
        IWinConditionChecker winChecker,
        IGameStateMachine stateMachine)
    {
        _sceneLoader = sceneLoader;
        _input = input;
        _winChecker = winChecker;
        _stateMachine = stateMachine;
    }

    public void OnEnter()
    {
        _input.OnPausePressed += HandlePause;
        _winChecker.OnWinConditionMet += HandleVictory;
        _winChecker.OnLoseConditionMet += HandleGameOver;
        Time.timeScale = 1f;
    }

    public void OnExit()
    {
        _input.OnPausePressed -= HandlePause;
        _winChecker.OnWinConditionMet -= HandleVictory;
        _winChecker.OnLoseConditionMet -= HandleGameOver;
    }

    public void OnUpdate() { } // Game world updates itself — no polling needed here

    private void HandlePause() => _stateMachine.Enter<PausedState>();
    private void HandleVictory() => _stateMachine.Enter<VictoryState>();
    private void HandleGameOver() => _stateMachine.Enter<GameOverState>();
}
```

### PausedState
```csharp
public sealed class PausedState : IGameState
{
    private readonly IInputProvider _input;
    private readonly IGameStateMachine _stateMachine;
    private readonly IAudioService _audio;

    public PausedState(IInputProvider input, IGameStateMachine stateMachine, IAudioService audio)
    {
        _input = input;
        _stateMachine = stateMachine;
        _audio = audio;
    }

    public void OnEnter()
    {
        Time.timeScale = 0f; // Freeze game world
        _audio.SetVolume(AudioChannel.Music, 0.3f); // Duck music
        _input.OnPausePressed += HandleResume;
    }

    public void OnExit()
    {
        Time.timeScale = 1f;
        _audio.SetVolume(AudioChannel.Music, 1f);
        _input.OnPausePressed -= HandleResume;
    }

    public void OnUpdate() { }

    private void HandleResume() => _stateMachine.Enter<GameplayState>();
}
```

---

## Win/Lose Condition Architecture

Win and lose conditions are checked by a dedicated service — they must NEVER be hardcoded into `PlayerController`, `EnemyManager`, or any gameplay system directly.

```csharp
// IWinConditionChecker.cs
public interface IWinConditionChecker
{
    event Action OnWinConditionMet;
    event Action OnLoseConditionMet;
}

// KillAllEnemiesWinCondition.cs (example)
public sealed class KillAllEnemiesWinCondition : IWinConditionChecker, IDisposable
{
    private readonly IEnemyTracker _tracker;
    private readonly IHealthSystem _playerHealth;

    public event Action OnWinConditionMet;
    public event Action OnLoseConditionMet;

    public KillAllEnemiesWinCondition(IEnemyTracker tracker, IHealthSystem playerHealth)
    {
        _tracker = tracker;
        _playerHealth = playerHealth;

        _tracker.OnAllEnemiesDefeated += () => OnWinConditionMet?.Invoke();
        _playerHealth.OnDied += () => OnLoseConditionMet?.Invoke();
    }

    public void Dispose() { /* unsubscribe */ }
}
```

---

## Session Data Management

Persist session-level data that must survive state transitions but reset between game sessions:

```csharp
// ISessionData.cs
public interface ISessionData
{
    int Score { get; }
    int EnemiesDefeated { get; }
    float SessionDuration { get; }
    void AddScore(int amount);
    void Reset();
}
```

Register `SessionData` in the `GameBootstrapper` — reset it explicitly in the `BootState.OnEnter()` call at the start of each new session.


---

## Bootstrapping the State Machine (Manual DI)

The `GameStateMachine` and all states are plain C# classes — they require no framework to wire up.

```csharp
// GameBootstrapper.cs — executed first via Script Execution Order
public sealed class GameBootstrapper : MonoBehaviour
{
    // Scene-assigned Views that need injection
    [SerializeField] private PauseMenuView _pauseMenu;

    private IGameStateMachine _stateMachine;

    private void Awake()
    {
        // 1. Create services (persisted ones fetched from ServiceLocator)
        IInputProvider input = ServiceLocator.Get<IInputProvider>();
        IAudioService audio = ServiceLocator.Get<IAudioService>();
        ISceneLoader sceneLoader = ServiceLocator.Get<ISceneLoader>();
        IWinConditionChecker winChecker = new KillAllEnemiesWinCondition(...);
        ISessionData sessionData = new SessionData();

        // 2. Create states — each receives its dependencies via constructor
        var bootState     = new BootState(sceneLoader);
        var mainMenuState = new MainMenuState(sceneLoader, audio);
        var loadingState  = new LoadingState(sceneLoader);
        var gameplayState = new GameplayState(input, winChecker, sessionData);
        var pausedState   = new PausedState(input, audio);
        var victoryState  = new VictoryState(sceneLoader, sessionData);
        var gameOverState = new GameOverState(sceneLoader);

        // 3. Build the state machine — pass all states as a collection
        _stateMachine = new GameStateMachine(new IGameState[]
        {
            bootState, mainMenuState, loadingState,
            gameplayState, pausedState, victoryState, gameOverState
        });

        // 4. Inject state machine into Views
        _pauseMenu.Construct(_stateMachine);

        // 5. Start the game loop
        _stateMachine.Enter<BootState>();
    }

    private void Update() => _stateMachine?.Tick();
}
```

> **VContainer upgrade path**: When the bootstrapper grows unwieldy (many systems), each `new XxxState(...)` line maps 1:1 to a `builder.Register<XxxState>()` call. The migration is purely mechanical — state classes don't change at all.


---

## UI Reaction to State Changes

UI screens listen to `IGameStateMachine.OnStateChanged` — they must NEVER poll the state machine or call game logic. Inject via `Construct()` (manual) or `[Inject]` attribute (if VContainer is used):

```csharp
public sealed class PauseMenuView : MonoBehaviour
{
    private IGameStateMachine _stateMachine;

    // Called by GameBootstrapper
    public void Construct(IGameStateMachine stateMachine)
    {
        _stateMachine = stateMachine;
        _stateMachine.OnStateChanged += HandleStateChanged;
    }

    private void OnDisable()
    {
        if (_stateMachine != null)
            _stateMachine.OnStateChanged -= HandleStateChanged;
    }

    private void HandleStateChanged(GameStateType state)
    {
        gameObject.SetActive(state == GameStateType.Paused);
    }
}
```


---

## Cross-Skill References
- Input handling inside states → `skills/input-system`
- Scene loading during LoadingState → `skills/scene-management`
- Audio transitions between states → `skills/audio-architecture`
- State machine pattern selection → `skills/design-pattern-architect` (FSM section)
- Win/lose condition testing → `skills/test-driven-dev`
