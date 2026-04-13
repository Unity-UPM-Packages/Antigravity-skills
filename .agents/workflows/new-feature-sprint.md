---
description: Full feature lifecycle — scope gate, interface definition, pure logic, MonoBehaviour bridge, DI wiring, tests, and commit.
---

# Workflow: New Feature Sprint

Execute the complete lifecycle of a new feature — from requirement analysis to a clean commit — following the `create-feature` skill SOP.

---

## Step 1 — Clarify Requirements (Scope Gate)

Answer all 4 questions before writing any code:

```
1. SINGLE RESPONSIBILITY: What is the one thing this feature does?
   If the answer contains "and" → split into two separate features.

2. DATA BOUNDARY: What data does it read? Write? Who owns it?

3. EVENT SURFACE: Who needs to know when something happens inside this system?

4. DUPLICATION CHECK: Does a similar system already exist?
   → If yes: extend it instead of creating a new one.
```

If any answer is unclear → ask the user before proceeding.

---

## Step 2 — Verify Folder Structure

```
Scripts/
├── Core/<FeatureName>/           ← IXxxSystem.cs, XxxData.cs
├── Systems/<FeatureName>/        ← XxxSystem.cs, XxxConfig.cs
├── Views/<FeatureName>/          ← XxxView.cs, XxxAnimator.cs
└── Bootstrap/GameBootstrapper.cs ← wiring
```

Create any missing folders before writing files.

---

## Step 3 — Define the Interface Contract

Create `Scripts/Core/<FeatureName>/I<Name>System.cs`:

```csharp
/// <summary>
/// Defines the contract for the <Name> system.
/// </summary>
public interface I<Name>System
{
    event Action<DataType> OnSomethingHappened;
    bool TryDoSomething(DataType data);
}
```

---

## Step 4 — Define the Data Model

Create `Scripts/Core/<FeatureName>/<Name>Data.cs`:

```csharp
/// <summary>Immutable value object representing a single <Name> entry.</summary>
[Serializable]
public readonly struct <Name>Data
{
    // Required fields — prefer readonly struct for value objects
}
```

---

## Step 5 — Implement Pure Logic

Create `Scripts/Systems/<FeatureName>/<Name>System.cs`:

```csharp
/// <summary>
/// Implements <see cref="I<Name>System"/>.
/// Pure C# — no UnityEngine dependency except primitive data types.
/// </summary>
public sealed class <Name>System : I<Name>System
{
    // Constructor injection only — no FindObjectOfType
    // Zero heap allocations in hot paths
}
```

---

## Step 6 — Create the MonoBehaviour Bridge (View)

Create `Scripts/Views/<FeatureName>/<Name>View.cs`:

```csharp
/// <summary>
/// Unity-facing bridge for <see cref="I<Name>System"/>.
/// Responsible for subscribing to events and forwarding input to the system.
/// </summary>
public sealed class <Name>View : MonoBehaviour
{
    private I<Name>System _system;

    /// <summary>Injects the system dependency. Call from GameBootstrapper.</summary>
    public void Construct(I<Name>System system)
    {
        _system = system;
        // Subscribe to events here
    }

    private void OnDestroy()
    {
        // Unsubscribe all events here
    }
}
```

---

## Step 7 — Wire into GameBootstrapper

Open `Scripts/Bootstrap/GameBootstrapper.cs` and add:

```csharp
// Inside Awake():
var <name>System = new <Name>System(/* dependencies */);
_<name>View.Construct(<name>System);
```

If `GameBootstrapper` does not exist → create it following the template in `skills/create-feature`.

---

## Step 8 — Write Tests

Create `Assets/Tests/EditMode/<Name>SystemTests.cs`:

```csharp
public class <Name>SystemTests
{
    [Test]
    public void <Name>_HappyPath_ShouldSucceed()
    {
        // Arrange → Act → Assert
    }

    [Test]
    public void <Name>_EdgeCase_ShouldHandleGracefully()
    {
        // Null input, empty state, max capacity, etc.
    }
}
```

Run all tests. Fix any failures before moving to the next step.

---

## Step 9 — Self-Review Checklist

Before committing, verify:
- [ ] No `FindObjectOfType` in new code
- [ ] No `new` heap allocations inside `Update()`
- [ ] All event subscriptions have a corresponding unsubscribe in `OnDestroy`
- [ ] Interface was defined before the implementation
- [ ] All public members have XML `<summary>` comments in English
- [ ] All tests pass

---

## Step 10 — Commit

```bash
# Stage by logical unit
git add Scripts/Core/<FeatureName>/ Scripts/Systems/<FeatureName>/
git commit -m "feat(<featurename>): implement <Name>System with interface contract"

git add Scripts/Views/<FeatureName>/
git commit -m "feat(<featurename>): add <Name>View MonoBehaviour bridge"

git add Scripts/Bootstrap/GameBootstrapper.cs
git commit -m "chore(bootstrap): wire <Name>System into GameBootstrapper"

git add Assets/Tests/
git commit -m "test(<featurename>): add EditMode unit tests for <Name>System"
```

---

## Cross-Skill References
- Full SOP details → `skills/create-feature`
- Component design principles → `skills/modular-design`
- Test writing → `skills/test-driven-dev`
- Commit format → `rules/06-git-conventions.md`
