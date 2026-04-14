---
name: 3d-texturing
description: "[3D Artist] Use when creating textures, PBR material setups, texture channel packing, or managing texture atlases for 3D models."
---

# Skill: Texturing & Materials

## When to use this skill
- User needs texture specifications for a 3D model
- User asks about PBR maps, channel packing, or texture optimization
- User wants to set up materials in Unity (URP or Built-in)

## Step-by-Step Execution

### Step 1 — Texture Plan

```
MODEL: [Name]
TEXTURE RESOLUTION: [256 / 512 / 1024 — from poly budget table]
STYLE: [PBR Realistic / Stylized Handpaint / Flat Color]
MAPS NEEDED: [Listed below]
ATLAS: [Standalone / Part of atlas [atlas_name]]
```

### Step 2 — PBR Map Set

| Map | Channel | Content | Required? |
|---|---|---|---|
| **Albedo (Base Color)** | RGB | Surface color without lighting | ✅ Always |
| **Normal Map** | RGB (tangent space) | Surface detail bumps | ✅ For anything > flat |
| **Mask Map (Packed)** | RGBA | See channel packing below | ✅ For PBR |
| **Emission** | RGB | Self-illuminating areas | ⚪ If needed |

### Step 3 — Channel Packing (Critical for Mobile)
Pack multiple data into ONE texture to reduce GPU sampling:

```
Mask Map (single RGBA texture):
  R = Metallic      (0 = non-metal, 1 = metal)
  G = Occlusion     (AO — cavity shadows)
  B = Detail Mask   (or custom data)
  A = Smoothness    (inverse of roughness)
```

**Result**: 4 data maps → 1 texture sample. 75% bandwidth savings.

### Step 4 — Stylized / Handpaint Workflow
For non-PBR art styles:

| Map | Purpose | Notes |
|---|---|---|
| **Albedo** | Painted color + baked shadows | Lighting is hand-painted in |
| **Normal** | Optional — subtle depth only | Low-intensity or skip entirely |
| **Emission** | Glowing elements | For magic effects, eyes |

- No metallic/smoothness maps needed
- Shader: Use **Unlit** or **Toon shader** (not PBR Lit)
- Color palette: Hand-pick from art direction, not from photo reference

### Step 5 — Texture Atlas Organization

| Atlas Name | Resolution | Contents | Material |
|---|---|---|---|
| `atlas_env_town` | 1024×1024 | Houses, fences, crates, barrels | 1 shared material |
| `atlas_chr_enemies` | 1024×1024 | 3–4 enemy types | 1 shared material |
| `atlas_props_combat` | 512×512 | Weapons, shields, potions | 1 shared material |

### Step 6 — Unity Material Setup

**URP:**
```
Shader: Universal Render Pipeline/Lit (or Simple Lit / Unlit)
Base Map: albedo_[name].png
Normal Map: normal_[name].png
Mask Map: mask_[name].png (assign channels accordingly)
Render Face: Front Only
```

**Built-in:**
```
Shader: Mobile/Bumped Diffuse (or Standard for hero assets)
Albedo: albedo_[name].png
Normal Map: normal_[name].png
Metallic/Smoothness: From mask map Alpha channel
```

## Best Practices
- Always keep source files (PSD/XCF) — export final PNG for Unity
- Mip Maps: ✅ Enable for 3D textures, ❌ Disable for UI sprites
- Compression: ASTC 4×4 for high quality, ASTC 6×6 for low
- Name textures: `[type]_[model]_[map].png` (e.g., `albedo_knight_body.png`)

---

## 2D Artist Handoff Protocol

### When 3D Artist works independently
| Texture type | Method | 2D needed? |
|---|---|---|
| Normal Map | Bake from high-poly in Blender | ❌ No |
| AO / Mask Map | Bake or procedural nodes | ❌ No |
| PBR Realistic albedo | Procedural (Blender Shader Nodes) | ❌ No |
| Toon / Stylized flat color | Procedural color ramp + noise | ❌ No |
| Library textures | Substance / Poly Haven assets | ❌ No |

### When to request 2D Artist
| Style | Signal | Action |
|---|---|---|
| **Hand-painted cartoon** (Clash of Clans, Hay Day) | Albedo must have brush strokes, painted shadows, outlines | 🎨 Request 2D Artist |
| **Illustrated / concept-art style** | Characters look like 2D illustrations on 3D form | 🎨 Request 2D Artist |
| **Custom icon / emblem on UV** (clan badge, weapon engrave) | Requires original artwork | 🎨 Request 2D Artist |

### How to hand off to 2D Artist
When requesting hand-painted albedo from the 2D Artist role:

**Step 1 — Provide UV Template**
```
1. Finish UV unwrap in Blender
2. UV Editor → Overlays → Export UV Layout
3. Format: PNG, Size: [Target Res], Fill Opacity: 0.25
4. Output: uv_[model]_[part]_template.png
```

**Step 2 — Write Albedo Brief**

```markdown
## Albedo Paint Request: [Model Name]

UV Template: uv_[model]_template.png
Canvas Size: [512 / 1024 / 2048]
Art Style: [Describe — e.g., "Clash of Clans-style hand painted, warm palette"]
Color References: [From GD art direction or mood board]

### Paint Instructions
| UV Region | Description | Color Hint |
|---|---|---|
| Top-left | Body armor | Dark steel blue, rivets |
| Center | Chest emblem | Gold, hand-drawn |
| Bottom | Boots | Worn brown leather |

### DO Nots
- No photographic textures — fully painted
- No gradients from photo tools — use brush strokes
- Shadows must be hand-painted into the albedo (no baked AO from 3D)
```

**Step 3 — Receive & Import**
```
1. Receive: albedo_[model]_[part].png from 2D Artist
2. Import to Blender → assign to material Base Color
3. Verify UV alignment in viewport
4. Re-bake Normal Map if needed (high-poly sculpt unchanged)
5. Export FBX + textures together to Unity
```
