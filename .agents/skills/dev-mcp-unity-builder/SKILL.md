---
name: dev-mcp-unity-builder
description: Use when the user provides any prompt, image, wireframe, or text description of UI screens, GameObjects, or Prefabs — and wants the AI to automatically create them in the Unity Editor via any available MCP tool.
---

# Skill: Prompt → Unity Builder via MCP

## Capability Overview
The agent accepts any form of input — image wireframe, screenshot, or plain text prompt — and autonomously creates the corresponding Unity objects in the active scene. This covers both uGUI UI screens and regular 3D/2D GameObjects or Prefabs. The agent handles MCP tool discovery, object creation, component attachment, script generation (if needed), hierarchy structuring, property configuration, error recovery, and fallback — end to end.

---

## Phase 0 — MCP Tool Discovery (Always First)

Never hardcode or assume a specific MCP tool name. Always discover what's available.

### 0.1 Discovery Steps
```
1. List all currently available MCP tools in the session
2. If user specified an MCP in their prompt → use that directly (skip filtering)
3. Otherwise: filter for Unity-capable tools using keyword recognition below
4. Map discovered tools to operation buckets
5. If no Unity-capable tool found → immediately switch to Editor Script fallback (Phase 7)
```

### 0.2 Unity Capability Keyword Recognition
A tool is **Unity-capable** if its name or description contains:

| Bucket | Keywords |
|---|---|
| **HIERARCHY** | `gameobject`, `create_object`, `hierarchy`, `scene`, `instantiate`, `spawn` |
| **COMPONENT** | `component`, `add_component`, `attach`, `script`, `monobehaviour` |
| **PROPERTY** | `property`, `set_value`, `field`, `transform`, `position`, `rotation` |
| **ASSET** | `prefab`, `asset`, `save_prefab`, `scriptableobject` |
| **GENERAL** | `unity`, `editor`, `inspector` |

### 0.3 User-Specified MCP
If prompt contains *"dùng MCP X"* / *"use [name]"* / *"via [tool]"* → skip discovery, use that tool directly. On failure → report specific error and ask whether to try another MCP or fall back to Editor Script.

### 0.4 Discovery Report (show before executing)
```
[MCP Discovery]
✅ Unity-capable tools found: [tool names]
  HIERARCHY → [tool]
  COMPONENT → [tool]
  PROPERTY  → [tool]
  ASSET     → [tool]
Proceeding...

── OR ──

[MCP Discovery]
❌ No Unity-capable MCP tools found.
Switching to Editor Script fallback.
```

---

## Phase 1 — Input Classification

### 1.1 Classify the Request

| Input | Examples | Route |
|---|---|---|
| Image / wireframe | Screenshot, Figma export, sketch | Analyze visually → Phase 2 |
| Text (UI) | "A shop screen with grid and buy button" | Parse → Phase 2 |
| Text (GameObject) | "An enemy with Rigidbody and HealthSystem" | Parse → Phase 3 |
| Text (Prefab) | "A chest that opens, plays sound, drops loot" | Parse → Phase 3 + Phase 4 |
| Mixed | "Player with a health bar above their head" | Both pipelines |

### 1.2 Route Decision
```
Primary output is Canvas/UI? → UI Pipeline (Phase 2)
Primary output is 3D/2D GameObject? → GameObject Pipeline (Phase 3)
Both? → UI Pipeline first, then GameObject Pipeline
```

### 1.3 Object Plan (produce before any MCP command)
```
[OBJECT PLAN]
Type: UI Screen | GameObject | Prefab
Name: ...

Hierarchy:
  Parent
  └── Child (Component, Component)
  └── Child (Component)

Scripts to attach:
  ✅ ExistingScript.cs  → found at Scripts/Systems/
  ⚙️ MissingScript.cs  → not found → generate (Phase 4)
```

---

## Phase 2 — UI Pipeline (Canvas / uGUI)

### 2.1 Canvas Partitioning
Assign every element to a Canvas branch before creating anything:

| Canvas | Content | Rebuilds When |
|---|---|---|
| `Canvas_Background` | Static art, decorations | Never |
| `Canvas_Main` | Menus, panels, dialogs | Screen opens/closes |
| `Canvas_HUD` | Health, timers, counters | Every data update |
| `Canvas_Overlay` | Popups, modals, loading | Show/hide |

### 2.2 Anchor Mapping

| Container Role | AnchorMin | AnchorMax | Safe Zone |
|---|---|---|---|
| Full-screen bg | (0,0) | (1,1) | None |
| Top header | (0,1) | (1,1) | `+= SafeArea.top` |
| Bottom action bar | (0,0) | (1,0) | `+= SafeArea.bottom` |
| Center popup | (0.5,0.5) | (0.5,0.5) | None |
| Scrollable area | stretch inside parent | — | Resp. header/footer |

`CanvasScaler` → `Scale With Screen Size`, ref resolution `1080×1920` or `1920×1080`.

### 2.3 Component Rules

| Need | Component | Rule |
|---|---|---|
| Any text | `TextMeshProUGUI` | Never use legacy `Text` |
| Decorative image | `Image` | `raycastTarget = false` |
| Rectangular clip | `RectMask2D` | Preferred over Mask |
| Scrollable list | `ScrollRect` + `VerticalLayoutGroup` | Pool if count > 10 |
| Grid | `ScrollRect` + `GridLayoutGroup` | Pool always |
| Button | `Button` + `TextMeshProUGUI` | Disable `Navigation` for mobile |

---

## Phase 3 — GameObject Pipeline

### 3.1 Component Manifest

```
[GAMEOBJECT PLAN]
Name: Enemy_Slime
Primitive: Capsule

Components:
  ✅ CapsuleCollider  → built-in
  ✅ Rigidbody        → built-in (freeze rotation XZ)
  ✅ Animator         → built-in
  ⚙️ HealthSystem.cs → check → attach or generate
  ⚙️ EnemyController → check → attach or generate

Children:
  └── HealthBarCanvas (Canvas, WorldSpace)

Tag: "Enemy" | Layer: "Enemy"
```

### 3.2 Component Classification

| Type | Agent Action |
|---|---|
| Built-in (Rigidbody, Collider, Animator) | Add directly via MCP |
| Project script — found in `Scripts/` | Add via MCP |
| Project script — not found | → Phase 4: Generate first |
| ScriptableObject config | Create `.asset`, wire as field |

---

## Phase 4 — Script Generation (Missing Scripts)

When a described script doesn't exist in the project:

1. **Search first**: check `Scripts/Systems/`, `Scripts/Views/`, `Scripts/Core/`
2. **If not found**: generate a skeleton following `skills/create-feature` SOP

```csharp
// HealthSystem.cs — generated skeleton
// Path: Scripts/Systems/Health/HealthSystem.cs
public sealed class HealthSystem : MonoBehaviour, IHealthSystem
{
    [SerializeField] private float _maxHealth = 100f;
    private float _currentHealth;

    public event Action OnDied;
    public event Action<float> OnHealthChanged; // normalized 0–1

    private void Awake() => _currentHealth = _maxHealth;

    public void TakeDamage(float amount)
    {
        _currentHealth = Mathf.Max(0f, _currentHealth - amount);
        OnHealthChanged?.Invoke(_currentHealth / _maxHealth);
        if (_currentHealth <= 0f) OnDied?.Invoke();
    }
}
```

**Rules:**
- MonoBehaviour scripts → `Scripts/Views/` or `Scripts/Systems/`
- Pure logic → `Scripts/Core/` (no MonoBehaviour)
- Always generate the interface alongside the implementation
- Wait for compilation before attaching via MCP

---

## Phase 5 — MCP Execution Protocol

Using the tools discovered in Phase 0, execute in this strict order:

```
For each object (parent before child):
  1. create_gameobject(name, primitiveType)
  2. set_parent(child, parent)
  3. set_transform(position, rotation, scale)
  4. add_component(builtInTypes)          ← built-ins first
  5. set_component_property(...)
  6. [UI only] set_rect_transform(anchor, pivot, offsets)
  7. add_component(customScriptName)      ← after script compiled
  8. set_component_property(script, field, targetObject)  ← wiring
```

### Command Sequencing Rules
1. **Parent before child** — never create a child before its parent exists
2. **Add component before setting its properties**
3. **Verify after each major step** — query hierarchy to confirm success
4. **Batch related commands** — group operations on the same GameObject

### Error Handling
```
After each command:
  → Query: "Does [GameObject] exist?" — if NO: retry
  → Query: "Does [Component] exist on [GO]?" — if NO: re-add
  → Query: "Is [property] = [expected]?" — if NO: re-set

After 2 consecutive failures on the same command → Phase 6 (Editor Script)
```

---

## Phase 6 — Fallback: Editor Script

When MCP is unavailable or fails, generate a Unity Editor `MenuItem` script:

```csharp
[MenuItem("Tools/Antigravity/Build ShopScreen")]
public static void BuildShopScreen()
{
    var canvasGO = new GameObject("Canvas_Shop");
    var canvas = canvasGO.AddComponent<Canvas>();
    canvas.renderMode = RenderMode.ScreenSpaceOverlay;

    var scaler = canvasGO.AddComponent<CanvasScaler>();
    scaler.uiScaleMode = CanvasScaler.ScaleMode.ScaleWithScreenSize;
    scaler.referenceResolution = new Vector2(1080, 1920);

    canvasGO.AddComponent<GraphicRaycaster>();
    // ... continue per Object Plan

    string path = "Assets/Prefabs/UI/ShopScreen.prefab";
    PrefabUtility.SaveAsPrefabAsset(canvasGO, path);
    Debug.Log($"[Antigravity] Built ShopScreen → {path}");
}
```

- Save to `Assets/Editor/Generated/Build[Name].cs`
- Instruct user: *"Run Tools → Antigravity → Build [Name] in Unity"*
- After user runs it, offer to resume via MCP if one becomes available

---

## Phase 7 — Validation Checklist

**For UI:**
- [ ] Canvas branches isolated correctly (Background / Main / HUD / Overlay)
- [ ] Decorative Images: `raycastTarget = false`
- [ ] CanvasScaler: `Scale With Screen Size`, correct reference resolution
- [ ] Safe area offsets on edge elements
- [ ] All text: `TextMeshProUGUI` (no legacy `Text`)
- [ ] Saved as `.prefab` in `Assets/Prefabs/UI/`

**For GameObjects:**
- [ ] All components attached (built-in + scripts)
- [ ] SerializedField references wired
- [ ] Correct Tag and Layer
- [ ] Saved as `.prefab` in `Assets/Prefabs/<Category>/`

**For generated scripts:**
- [ ] Compiles without errors before attachment
- [ ] Interface created alongside implementation
- [ ] Tests drafted per `skills/test-driven-dev`

---

## Example Prompts

| Prompt | Pipeline |
|---|---|
| `[image] "Build this shop UI"` | UI |
| `"HUD with health bar, ammo counter, minimap"` | UI |
| `"Enemy: CapsuleCollider, Rigidbody, HealthSystem"` | GO |
| `"Treasure chest: opens on interact, plays audio, spawns loot"` | GO + Script Gen |
| `"Player with world-space health bar above head"` | GO + UI (mixed) |
| `"Main menu: Play, Settings, Quit buttons"` | UI |
| `"Use [MCP name] to build this prefab"` | GO, user-specified MCP |

---

## Cross-Skill References
- Script generation SOP → `skills/create-feature`
- Canvas performance rules → `skills/ui-performance`
- Editor Script patterns → `skills/editor-scripting`
- Responsive anchor rules → `rules/05-responsive-ugui.md`
- Binding UI to game logic → `skills/modular-design`
