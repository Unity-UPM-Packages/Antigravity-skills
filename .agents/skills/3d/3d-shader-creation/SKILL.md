---
name: 3d-shader-creation
description: "[3D Artist] Use when creating custom shaders — Shader Graph (URP), Surface Shaders (Built-in), or raw HLSL/CG. Covers toon shaders, dissolve effects, outlines, water, and mobile-optimized shader patterns."
---

# Skill: Shader Creation

## When to use this skill
- User needs a custom shader (toon, dissolve, outline, hologram, water, etc.)
- User asks about Shader Graph nodes or HLSL code
- User wants to optimize an existing shader for mobile

## Decision Tree

| Need | Pipeline | Approach |
|---|---|---|
| Simple custom look | URP | **Shader Graph** — visual, fast iteration |
| Complex/optimized effect | URP | **HLSL in URP** — full control, SRP Batcher |
| Legacy project shader | Built-in | **Surface Shader** (CG) — auto lighting |
| Maximum control (Built-in) | Built-in | **Vert/Frag shader** (CG/HLSL) — manual everything |

## Common Shader Recipes

### Toon / Cel Shader
```
Concept: Quantize lighting into discrete bands instead of smooth gradient
Key nodes/code:
  1. Calculate NdotL (normal · light direction)
  2. Step or SmoothStep into 2–4 bands
  3. Multiply by albedo color
  4. Add rim lighting (Fresnel) for edge glow
Performance: Low — single texture sample + math
```

### Dissolve Effect
```
Concept: Use noise texture to progressively hide the mesh
Key nodes/code:
  1. Sample noise texture
  2. Compare noise value against _DissolveAmount (0→1)
  3. Clip pixels below threshold
  4. Add emission at dissolve edge (hot glow)
Performance: Medium — 1 extra texture sample + clip
```

### Outline (Inverted Hull)
```
Concept: Render mesh twice — once normal, once inflated with back-face only
Key nodes/code:
  Pass 1: Normal render
  Pass 2: Cull Front, Vertex += normal * _OutlineWidth, Color = _OutlineColor
Performance: Medium — doubles draw call for outlined object
```

### Mobile Water
```
Concept: Animated UV scrolling + depth-based transparency
Key nodes/code:
  1. Scroll UV with _Time
  2. Sample normal map for ripples
  3. Depth fade at edges (camera depth texture)
  4. Fresnel for edge foam
Performance: Medium-High — uses depth texture (verify GPU support)
```

## Shader Graph Best Practices (URP)
- **Max 50 nodes** for mobile shaders
- Use **Sub Graphs** for reusable logic (noise, rim, UV scroll)
- Set **Precision** to Half where possible (Properties → Half)
- Always preview on target device — editor preview ≠ mobile result
- Enable **SRP Batcher** compatibility in shader settings

## HLSL/CG Coding Conventions
```hlsl
// Naming: _PascalCase for properties, camelCase for locals
// Precision: half > float on mobile
half4 frag(v2f i) : SV_Target
{
    half4 baseColor = tex2D(_BaseMap, i.uv);
    half ndotl = saturate(dot(i.normalWS, _MainLightDirection));
    return baseColor * ndotl;
}
```

- Use `TEXTURXE2D` + `SAMPLER` macros (URP) instead of `tex2D` (Built-in)
- Avoid `sin`, `cos`, `pow` in fragment shader hot paths — precompute in vertex
- Use `[branch]` attribute for expensive conditional blocks (avoids both-path execution)
- Built-in: Add `noforwardadd noshadow` pragmas for single-pass mobile shaders

## Shader Debugging
| Visualization | Purpose | How |
|---|---|---|
| Normal direction | Check normal map orientation | Output `normal * 0.5 + 0.5` |
| UV layout | Check UV distortion | Output `half4(uv, 0, 1)` |
| Overdraw | Find expensive overlap | Scene view → Overdraw mode |
| Depth | Check depth buffer | Output depth as grayscale |
