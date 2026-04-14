---
trigger: always_on
glob:
description: "[3D Artist] Polygon budgets per asset category, clean topology requirements, UV unwrapping standards, and naming conventions for mobile 3D assets."
---

# Rule 3D-01: 3D Modeling Standards

## Overview
Every 3D model must be production-ready for mobile: optimized poly count, clean topology, efficient UVs, and consistent naming. A beautiful model that kills FPS is a failed model.

## 1. Polygon Budget (Mobile)

| Asset Category | Triangle Limit | Texture Resolution | LOD Required? |
|---|---|---|---|
| Hero Character (player) | 5,000 – 10,000 tris | 512 or 1024 | Yes (3 LODs) |
| Main NPC / Boss | 5,000 – 8,000 tris | 512 or 1024 | Yes (2 LODs) |
| Standard Enemy | 2,000 – 5,000 tris | 256 or 512 | Yes (2 LODs) |
| Environment Prop (small) | 100 – 500 tris | 128 or 256 | No |
| Environment Prop (medium) | 500 – 2,000 tris | 256 or 512 | Optional |
| Environment Prop (large) | 2,000 – 5,000 tris | 512 | Yes |
| Vehicle / Mount | 3,000 – 8,000 tris | 512 | Yes (2 LODs) |
| Weapon / Equipment | 500 – 2,000 tris | 256 or 512 | No |
| Pickup / Collectible | 50 – 300 tris | 128 | No |

> ⚠️ **Total scene budget**: Aim for < 100K visible triangles per frame on low-end mobile (Adreno 506 / Mali-G52 tier).

## 2. Topology Requirements

### Clean Mesh Rules
- **Quads preferred** for deformable meshes (characters, cloth). Tris allowed for static props
- **No N-gons** (5+ sided faces) — they cause unpredictable triangulation
- **No floating vertices** or disconnected geometry (unless intentional for VFX)
- **No overlapping faces** — causes z-fighting and lighting artifacts
- **No flipped normals** — check before every export

### Edge Flow (Deformable Meshes)
- Edge loops must follow muscle/joint flow for clean deformation
- Minimum 3 edge loops around joints (elbow, knee, shoulder)
- Face density concentrated at deformation zones, sparse on flat surfaces

## 3. UV Standards

### UV Unwrap Rules
- **No overlapping UVs** (unless mirrored for symmetry optimization)
- **Texel density**: Consistent across the model — don't waste half the UV on a boot sole
- **Padding**: Minimum 4px between UV islands at target resolution (prevents bleed)
- **UV Space utilization**: Aim for ≥ 75% UV space usage
- **UV Channel 0**: Albedo/Normal/etc. mapping
- **UV Channel 1** (if needed): Lightmap UVs — non-overlapping, no stretch

### Atlas Considerations
- Props sharing the same material should share a single texture atlas
- Atlas size: POT (256, 512, 1024, 2048)
- Group by render frequency: objects visible together share the atlas

## 4. Naming Convention

### Files
```
[type]_[category]_[name]_[variant].fbx
```
Examples:
- `chr_player_knight_default.fbx`
- `env_prop_barrel_broken.fbx`
- `wpn_sword_flame.fbx`
- `veh_horse_armored.fbx`

### Type Prefixes
| Prefix | Type |
|---|---|
| `chr_` | Character / Creature |
| `env_` | Environment / Architecture |
| `prop_` | Props / Interactables |
| `wpn_` | Weapon |
| `veh_` | Vehicle / Mount |
| `vfx_` | VFX mesh |

### Blender Internal
- Objects: `[Name]_mesh`, `[Name]_armature`, `[Name]_collider`
- Materials: `mat_[name]_[part]` (e.g., `mat_knight_body`)
- Bones: Follow Unity Humanoid naming for auto-retarget compatibility
