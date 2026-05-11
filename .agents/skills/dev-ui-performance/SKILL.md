---
name: dev-ui-performance
description: Use when optimizing uGUI Canvas layouts, batching elements to reduce draw calls, and preventing UI raycast overhead spikes. Also activates proactively when the AI detects dynamic UI elements sharing a Canvas with static elements, nested LayoutGroups deeper than 2 levels, or Image components with raycastTarget enabled on non-interactive decorations.
---

# Skill: UI Performance Optimization (uGUI)

---

## Core Principles

### 1. Canvas Partitioning Strategy (Most Impactful)
A Canvas rebuilds its **entire** geometry mesh whenever any child element changes. Incorrect Canvas segmentation is the single most common mobile UI performance killer.

| Canvas Branch | Content | Rebuild Frequency |
|---|---|---|
| `Canvas_Background` | Static art, non-interactive decorations | Never (after init) |
| `Canvas_Main` | Menu panels, settings, shop screens | Per panel open/close |
| `Canvas_HUD` | Health bars, timers, ammo counters | Every frame (isolate!) |
| `Canvas_Overlay` | Popups, modals, tutorial hints | On show/hide only |

**Rule**: Any element that updates more than once per second **must** be on its own Canvas or a child Canvas component. Failure forces the entire parent Canvas to rebuild every frame.

```csharp
// If a HealthBar updates constantly, give it its own Canvas
// DO NOT put it on the same Canvas as static menu art
[RequireComponent(typeof(Canvas))] // Each dynamic element gets its own Canvas
public sealed class HealthBarView : MonoBehaviour { ... }
```

### 2. Draw Call Batching (Sprite Atlas)
Unity batches UI elements that share the same material and texture. Crossing texture boundaries breaks the batch.

- **Mandate Sprite Atlases** for all UI icons, buttons, and panel assets
- Group atlases by usage context: `Atlas_HUD`, `Atlas_MainMenu`, `Atlas_Shop`
- Never mix atlas sprites on the same Canvas with single-texture sprites — it breaks batching
- Use `URP 2D Renderer` sprite atlases for consistency with the rendering pipeline

### 3. Raycast Overhead Reduction
Every `Image` and `Text` component has `Raycast Target` enabled by default. This means every touch event iterates through every enabled raycaster — including decorative elements.

**Rule**: Set `raycastTarget = false` on ALL of the following:
- Background images
- Decorative icons and borders
- Text labels that are read-only
- Progress bar fill images

Only enable `Raycast Target` on **interactive elements**: `Button`, `Toggle`, `Slider`, `InputField`.

```csharp
// Enforce programmatically in a PostProcessor or Editor Tool
public static void DisableNonInteractiveRaycasts(Transform root)
{
    foreach (var img in root.GetComponentsInChildren<Image>())
    {
        if (img.GetComponent<Button>() == null && img.GetComponent<Toggle>() == null)
            img.raycastTarget = false;
    }
}
```

### 4. Masking — Use the Right Type
| Need | Component | Reason |
|---|---|---|
| Rectangular crop (scroll view viewport) | `RectMask2D` | No stencil buffer modification, GPU-cheaper |
| Non-rectangular or circular clip | `Mask` + `Image` | Writes to depth stencil — higher cost |
| Soft edges / feathering | Shader-based mask | `RectMask2D` with softness parameter |

Never use `Mask` for a simple rectangular crop. The stencil write cost is unnecessary.

### 5. ScrollRect & List Performance (Object Pooling Mandatory)
A `ScrollRect` with > 10 items that instantiates all items at once is a guaranteed performance problem on mobile.

```csharp
// Use a pooled ScrollRect — PooledScrollView pattern
public sealed class InventoryScrollView : MonoBehaviour
{
    [SerializeField] private GameObject _itemPrefab;
    [SerializeField] private RectTransform _content;

    // Pool pre-allocated item cells
    private readonly Queue<InventoryItemCell> _pool = new();
    private readonly List<InventoryItemCell> _active = new();

    public void PopulateWith(IReadOnlyList<ItemData> items)
    {
        ReturnAllToPool();

        foreach (ItemData item in items)
        {
            InventoryItemCell cell = GetFromPool();
            cell.Bind(item); // Set data without instantiating
            _active.Add(cell);
        }
    }

    private InventoryItemCell GetFromPool()
    {
        if (_pool.Count > 0)
        {
            var cell = _pool.Dequeue();
            cell.gameObject.SetActive(true);
            return cell;
        }
        return Instantiate(_itemPrefab, _content).GetComponent<InventoryItemCell>();
    }

    private void ReturnAllToPool()
    {
        foreach (var cell in _active)
        {
            cell.gameObject.SetActive(false);
            _pool.Enqueue(cell);
        }
        _active.Clear();
    }
}
```

### 6. Layout Group Performance
`LayoutGroup` components (`HorizontalLayoutGroup`, `VerticalLayoutGroup`, `GridLayoutGroup`) trigger a full layout rebuild when any child changes. Rules:
- **Disable** `Layout Groups` once layout is finalized — use `LayoutRebuilder.ForceRebuildLayoutImmediate()` then remove
- Never nest multiple `LayoutGroup` components inside each other (nested rebuild cascade)
- For fixed grids, pre-calculate sizes and set `RectTransform` manually

### 7. TextMeshPro Optimization
- Use shared font atlases — avoid creating per-text font assets
- Enable **Sprite Atlas** mode in TMP Settings for inline icon rendering
- Set `enableWordWrapping = false` for fixed-length HUD text (timers, scores) to skip text reflow calculation
- Use `TMP_Text.SetText(string format, int value)` overload to avoid string allocations:

```csharp
// ❌ Allocates a new string every frame
_scoreText.text = $"Score: {_score}";

// ✅ Zero allocation
_scoreText.SetText("Score: {0}", _score);
```

### 8. GraphicRaycaster Optimization
- Disable `GraphicRaycaster` on non-interactive Canvases entirely (background, HUD visuals)
- Set `Blocking Mask` to only include layers that have geometry (avoid unnecessary physics overlap tests)
- Consider disabling the raycaster when a popup is open and the underlying screen is not interactive

---

## UI Performance Audit Checklist
Before shipping any UI screen:
- [ ] Dynamic elements (counters, bars) are on an isolated Canvas
- [ ] All decorative Images/Text have `raycastTarget = false`
- [ ] `RectMask2D` used instead of `Mask` for rectangular viewports
- [ ] ScrollRect lists use object pooling if item count can exceed 10
- [ ] Nested `LayoutGroup` components eliminated
- [ ] Sprite Atlas applied to all UI art (no single-texture sprites on same Canvas)
- [ ] TMP `SetText` overloads used in frequently-updating labels
- [ ] Non-interactive Canvases have `GraphicRaycaster` disabled

---

## Cross-Skill References
- UI construction via MCP → `skills/prompt-to-mcp-builder`
- Responsive anchor decisions → `rules/05-responsive-ugui.md`
- Profiling canvas rebuild cost → `skills/unity-profiler-mind`
