---
name: test-driven-dev
description: Use when writing tests for new or existing logic, validating system behaviour, or setting up Unity NUnit EditMode and PlayMode test suites. Also activates proactively when the AI implements any pure C# logic that has no accompanying test file, or detects untested edge cases in existing code.
---

# Workflow: Test-Driven AI (TDD)

## Objective
Enforce autonomous verification of mechanics. The AI must prove its code functions correctly via the Unity Test Framework **before** requesting manual human review. The user's role is purely "Reviewer" — not debugger.

---

## When to Apply TDD

| Scenario | Test Type Required |
|---|---|
| Pure logic / math (damage calc, economy formula) | EditMode — fast, no scene |
| State machine transitions | EditMode with mock dependencies |
| MonoBehaviour lifecycle behavior | PlayMode in minimal test scene |
| Physics / collision / time-based logic | PlayMode |
| Repository / save-load flows | EditMode with mock file system |
| UI event binding verification | PlayMode |

---

## Execution Sequence

### Step 1 — Write Tests First (Red Phase)
Before any production code, write NUnit assertions that define the expected behavior. Tests should fail at this point.

```csharp
// Tests/EditMode/HealthSystemTests.cs
using NUnit.Framework;

[TestFixture]
public sealed class HealthSystemTests
{
    private HealthSystem _sut; // System Under Test

    [SetUp]
    public void SetUp() => _sut = new HealthSystem(maxHealth: 100);

    [Test]
    public void TakeDamage_ReducesCurrentHealth()
    {
        _sut.TakeDamage(30);
        Assert.AreEqual(70, _sut.CurrentHealth);
    }

    [Test]
    public void TakeDamage_BeyondMax_ClampsAtZero()
    {
        _sut.TakeDamage(999);
        Assert.AreEqual(0, _sut.CurrentHealth);
    }

    [Test]
    public void TakeDamage_FiresOnDied_WhenHealthReachesZero()
    {
        bool diedFired = false;
        _sut.OnDied += () => diedFired = true;

        _sut.TakeDamage(100);

        Assert.IsTrue(diedFired, "OnDied event was not fired.");
    }

    [Test]
    public void Heal_DoesNotExceedMaxHealth()
    {
        _sut.TakeDamage(50);
        _sut.Heal(999);
        Assert.AreEqual(100, _sut.CurrentHealth);
    }
}
```

### Step 2 — Implement Logic (Green Phase)
Write the minimum production code to make all tests pass. No over-engineering at this stage.

```csharp
// Systems/Health/HealthSystem.cs
public sealed class HealthSystem : IHealthSystem
{
    private int _currentHealth;
    private readonly int _maxHealth;

    public int CurrentHealth => _currentHealth;
    public int MaxHealth => _maxHealth;
    public event Action OnDied;
    public event Action<int> OnHealthChanged;

    public HealthSystem(int maxHealth)
    {
        _maxHealth = maxHealth;
        _currentHealth = maxHealth;
    }

    public void TakeDamage(int amount)
    {
        _currentHealth = Mathf.Max(0, _currentHealth - amount);
        OnHealthChanged?.Invoke(_currentHealth);

        if (_currentHealth == 0)
            OnDied?.Invoke();
    }

    public void Heal(int amount)
    {
        _currentHealth = Mathf.Min(_maxHealth, _currentHealth + amount);
        OnHealthChanged?.Invoke(_currentHealth);
    }
}
```

### Step 3 — Refactor (Clean Phase)
With passing tests as a safety net, refactor for clarity and performance without fear of regression.

### Step 4 — PlayMode Tests for MonoBehaviour Integration

```csharp
// Tests/PlayMode/HealthComponentPlayTests.cs
using System.Collections;
using NUnit.Framework;
using UnityEngine;
using UnityEngine.TestTools;

[TestFixture]
public sealed class HealthComponentPlayTests
{
    private GameObject _testGO;
    private HealthComponent _healthComponent;

    [UnitySetUp]
    public IEnumerator SetUp()
    {
        _testGO = new GameObject("TestPlayer");
        _healthComponent = _testGO.AddComponent<HealthComponent>();
        yield return null; // Wait for Awake/Start
    }

    [UnityTearDown]
    public IEnumerator TearDown()
    {
        Object.Destroy(_testGO);
        yield return null;
    }

    [UnityTest]
    public IEnumerator HealthComponent_DiesAfterLethalDamage()
    {
        bool isDead = false;
        _healthComponent.OnDied += () => isDead = true;

        _healthComponent.TakeDamage(9999);

        yield return null; // Allow events to propagate

        Assert.IsTrue(isDead);
    }
}
```

---

## Mocking with NSubstitute
Use `NSubstitute` to isolate the system under test from its dependencies:

```csharp
// Install: NSubstitute via NuGet or Unity Package
using NSubstitute;
using NUnit.Framework;

[TestFixture]
public sealed class PlayerControllerTests
{
    [Test]
    public void Player_Dies_WhenHealthSystemFiresOnDied()
    {
        // Arrange
        IHealthSystem mockHealth = Substitute.For<IHealthSystem>();
        var player = new PlayerController(mockHealth);

        // Act — simulate the event firing
        mockHealth.OnDied += Raise.Event<Action>();

        // Assert
        Assert.IsTrue(player.IsDead);
    }
}
```

---

## Test File Organization
```
Tests/
├── EditMode/
│   ├── HealthSystemTests.cs
│   ├── InventorySystemTests.cs
│   └── EconomyCalculatorTests.cs
└── PlayMode/
    ├── HealthComponentPlayTests.cs
    └── InputHandlerPlayTests.cs

# Each assembly needs its own .asmdef:
Tests/EditMode/Game.Tests.EditMode.asmdef
Tests/PlayMode/Game.Tests.PlayMode.asmdef
```

---

## Self-Correction Protocol
When a test fails after implementation:
1. Read the assertion error message precisely
2. Trace back to the exact line causing the failure
3. Fix the implementation — **never weaken the test** to make it pass
4. Re-run and confirm green before proceeding

---

## Minimum Test Coverage Targets
| System Type | Minimum Tests |
|---|---|
| Core logic class | 4 tests (happy path, edge cases, event firing, null inputs) |
| Repository / save | 3 tests (write, read, corrupted data fallback) |
| State machine | N tests = N state transitions minimum |
| UI Presenter | 2 tests (data binding update, event subscription cleanup) |

---

## Cross-Skill References
- Writing testable code structure → `skills/modular-design` (Interface Contracts section)
- Feature implementation alongside tests → `skills/create-feature` (Step 6)
- Mocking complex scene dependencies → use NSubstitute for pure logic; see `skills/test-driven-dev` (Mocking section)
