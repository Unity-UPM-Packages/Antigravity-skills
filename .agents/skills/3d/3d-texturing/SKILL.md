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
- Compression: ASTC 4×4 for high quality, ASTC 6×6 for mid, ASTC 8×8 for low
- Name textures: `[type]_[model]_[map].png` (e.g., `albedo_knight_body.png`)
