---
name: 3d-modeling
description: "[3D Artist] Use when creating 3D models in Blender — covers topology, edge flow, UV unwrapping, normal baking, and export-ready mesh preparation for Unity mobile."
---

# Skill: 3D Modeling (Blender)

## When to use this skill
- User asks to create or modify a 3D model
- User needs guidance on topology, UV layout, or mesh cleanup
- User mentions Blender modeling workflow

## Step-by-Step Execution

### Step 1 — Model Brief
Confirm before modeling:

```
MODEL: [Name]
TYPE: [Character / Environment / Prop / Weapon / Vehicle]
POLY BUDGET: [from 3d-01-modeling-standards]
TEXTURE RES: [256 / 512 / 1024]
DEFORMABLE: [Yes (needs clean edge flow) / No (static)]
LOD REQUIRED: [Yes (how many levels) / No]
REFERENCE: [Description or visual reference]
```

### Step 2 — Blender Workflow

| Phase | Actions | Key Shortcuts |
|---|---|---|
| **Block-out** | Primitive shapes → basic silhouette | `Shift+A` → Mesh |
| **Refine** | Add edge loops, shape details | `Ctrl+R` loop cut |
| **Topology cleanup** | Remove N-gons, fix quads, merge verts | `M` merge, `X` dissolve |
| **UV Unwrap** | Mark seams → Unwrap → Pack | `U` → Unwrap |
| **High-to-Low bake** | Sculpt detail → Bake normals to low-poly | Render → Bake |
| **Final check** | Normals, scale, origin, naming | `Shift+N` recalculate normals |

### Step 3 — Pre-export Checklist

- [ ] Apply all transforms (`Ctrl+A` → All Transforms)
- [ ] Origin at model base (feet for characters, center for props)
- [ ] Scale: 1 Blender unit = 1 Unity meter
- [ ] No hidden geometry or orphan data
- [ ] Materials assigned with correct naming (`mat_[name]`)
- [ ] UV Channel 0 clean (no overlaps except mirrored)
- [ ] Mesh named per `3d-01-modeling-standards` convention

### Step 4 — Export Settings (FBX)

```
Format: FBX
Scale: 1.0 (with "Apply Unit" checked)
Forward: -Z Forward
Up: Y Up
Apply Transform: ✅
Mesh:
  ✅ Apply Modifiers
  ✅ Tangent Space
  ❌ Loose Edges
  ❌ Custom Properties
Armature (if rigged):
  ✅ Only Deform Bones
  ❌ Add Leaf Bones
```

## Best Practices
- Model at final game scale from the start — never "scale later"
- Use mirror modifier for symmetric models (cuts UV work in half)
- Bevel edges that catch light (2–3 segments) for better shading without extra geo
- Keep mesh density proportional to screen importance
