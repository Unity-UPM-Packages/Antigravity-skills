---
name: 2d-asset-spec
description: "[2D Artist & UI/UX] Use when writing production specifications for sprites, icons, backgrounds, atlas sheets, or any 2D visual asset. Outputs artist-ready spec sheets."
---

# Skill: Asset Production Specification

## When to use this skill
- User needs to spec out sprites, icons, or backgrounds for production
- User asks "what size should this be?" or "how should I organize the sprite sheet?"
- Assets need to be prepared for Unity import with correct settings

## Step-by-Step Execution

### Step 1 — Asset Inventory
List all assets needed with classification:

| Asset | Type | Screen/Feature | Priority |
|---|---|---|---|
| [Name] | Sprite / Icon / BG / 9-Slice / Animation | [Where used] | MUST / NICE |

### Step 2 — Individual Asset Spec Card

```markdown
## Asset: [Asset Name]

| Field | Value |
|---|---|
| **Type** | [Sprite / Icon / Background / 9-Slice Panel / Atlas] |
| **Dimensions** | [Width × Height in pixels — POT if standalone] |
| **Format** | [PNG-24 / JPG / SVG] |
| **Alpha** | [Yes / No — premultiplied?] |
| **9-Slice** | [Yes/No — if Yes: L:__px R:__px T:__px B:__px] |
| **Atlas Target** | [Atlas name — e.g., "UI_Common_Atlas"] |
| **Compression** | [ASTC 4×4 / ASTC 6×6 / ASTC 8×8] |
| **Used In** | [Screen names where this appears] |
| **States Required** | [default / pressed / disabled / locked / etc.] |
| **Visual Reference** | [Description or mood reference] |
| **Naming** | [Following 2d-03 convention: category/screen-element-variant-state] |
```

### Step 3 — Sprite Sheet Specification (Multi-frame Assets)

```markdown
## Sprite Sheet: [Character/Effect Name]

| Field | Value |
|---|---|
| **Sheet Size** | [1024×1024 / 2048×2048 — POT] |
| **Frame Size** | [128×128 per frame] |
| **Frames Total** | [N frames] |
| **Layout** | [Grid: 8 columns × N rows] |
| **Animations** | [Listed below] |

### Animation Clips
| Clip Name | Frames | FPS | Loop? |
|---|---|---|---|
| idle | 1–8 | 12 | Yes |
| walk | 9–16 | 12 | Yes |
| attack | 17–24 | 24 | No |
| death | 25–30 | 12 | No |
```

### Step 4 — Atlas Organization Plan

| Atlas Name | Max Size | Contents | Estimated Fill |
|---|---|---|---|
| UI_Common | 2048×2048 | Buttons, icons, frames | ~70% |
| UI_Shop | 1024×1024 | Shop-specific elements | ~50% |
| Characters_Main | 2048×2048 | Player + 3 companions | ~80% |
| VFX_Combat | 512×512 | Hit effects, particles | ~60% |

### Step 5 — Unity Import Settings Recommendation

| Asset Type | Texture Type | Max Size | Compression | Filter | Packing Tag |
|---|---|---|---|---|---|
| UI Sprite | Sprite (2D) | 2048 | ASTC 4×4 | Bilinear | UI_Common |
| Character | Sprite (2D) | 2048 | ASTC 4×4 | Bilinear | Characters |
| Background | Default | 2048 | ASTC 6×6 | Bilinear | — |
| Icon | Sprite (2D) | 512 | ASTC 4×4 | Bilinear | UI_Icons |

## Best Practices
- All standalone textures MUST be POT (see `gd-03-production-specs`)
- Group frequently co-rendered sprites into the same atlas for draw call batching
- Always over-estimate atlas space by 20% for future additions
- Name files consistently before import — renaming inside Unity breaks .meta references
