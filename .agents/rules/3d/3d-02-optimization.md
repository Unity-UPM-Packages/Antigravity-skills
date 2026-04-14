---
trigger: always_on
glob:
description: "[3D Artist] LOD strategy, mesh optimization, draw call management, occlusion culling, and mobile GPU performance constraints."
---

# Rule 3D-02: 3D Optimization

## Overview
Mobile GPUs have severe limitations: limited VRAM, thermal throttling, and narrow fill-rate budgets. Every model, material, and effect must be designed for performance from day one.

## 1. LOD (Level of Detail) Strategy

### LOD Tiers
| LOD Level | Distance | Triangle Reduction | Usage |
|---|---|---|---|
| **LOD 0** | 0 – 10m | 100% (full detail) | Close-up, hero shots |
| **LOD 1** | 10 – 25m | 50% of LOD 0 | Mid-range gameplay |
| **LOD 2** | 25 – 50m | 25% of LOD 0 | Background, crowd |
| **LOD 3 / Cull** | 50m+ | Billboard or invisible | Not rendered |

### LOD Rules
- Characters with > 3,000 tris MUST have at least 2 LOD levels
- Environment props with > 1,000 tris SHOULD have LODs
- Use Blender's Decimate modifier for LOD generation, then manual cleanup
- LOD transitions: use cross-fade (dithering) to avoid pop-in

## 2. Mesh Optimization

### Merge Strategy
- Static props in the same area → **Mesh Combine** (reduces draw calls)
- Same material, same lighting = combine-able
- Don't combine objects that need independent culling (hiding behind walls)

### Vertex Count Reduction
- Remove unseen faces (bottom of ground-sitting objects, interior of closed shapes)
- Bake high-poly detail into Normal Maps rather than using geometry
- Use **alphatest cutout** for complex silhouettes (leaves, fences) instead of high-poly geometry

## 3. Draw Call Budget

| Platform Tier | Max Draw Calls/Frame | Max SetPass/Frame |
|---|---|---|
| Low-end mobile | 50 – 80 | 30 |
| Mid-range mobile | 100 – 150 | 50 |
| High-end mobile | 200 – 300 | 80 |

### Reduction Techniques
- **Material Atlasing**: Share one material across many objects
- **Static Batching**: Enable for non-moving objects
- **Dynamic Batching**: For small meshes < 300 vertices
- **GPU Instancing**: For repeated identical meshes (trees, rocks, grass)
- **SRP Batcher**: Enable for URP (compatible shaders only)

## 4. Occlusion & Culling
- Mark large static geometry as **Occluders**
- Mark small props as **Occludees**
- Use **frustum culling** (automatic in Unity) + **occlusion culling** (bake required)
- For open worlds: implement distance-based LOD + culling zones

## 5. Mobile GPU Constraints

| Constraint | Limit | Impact |
|---|---|---|
| Fill Rate | Limited — avoid overdraw | Transparent/alpha objects are expensive |
| VRAM | 1–3 GB shared | Texture compression critical (ASTC) |
| Bandwidth | Narrow memory bus | Large textures = slow reads |
| Thermal | Throttles after ~15min heavy load | Sustain < 60% GPU utilization |

### Performance Checklist
- [ ] Total visible tris per frame < 100K (low-end target)
- [ ] Draw calls < 80 (low-end target)
- [ ] No real-time shadows on low-end (use blob/decal shadows)
- [ ] Texture memory < 150MB total loaded
- [ ] No alpha-blended overlapping layers > 3 deep (overdraw)
