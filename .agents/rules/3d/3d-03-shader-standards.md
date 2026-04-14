---
trigger: model_decision
glob:
description: "[3D Artist] Shader standards for both URP and Built-in Render Pipeline, Shader Graph guidelines, HLSL coding rules, texture channel packing, and fill rate management."
---

# Rule 3D-03: Shader & Material Standards

## Overview
Shaders are the single biggest performance lever on mobile. A bad shader can halve your frame rate. This rule covers both **URP (Universal Render Pipeline)** and **Built-in Render Pipeline** to support all project configurations.

## 1. Render Pipeline Selection

### Decision Matrix
| Criteria | Built-in RP | URP |
|---|---|---|
| **Legacy projects** | ✅ Use this | Requires migration |
| **New projects (2021+)** | Consider migrating | ✅ Preferred |
| **Custom shader control** | Surface Shaders + CG/HLSL | Shader Graph + HLSL |
| **SRP Batcher** | ❌ Not available | ✅ Significant batching gains |
| **Shader Graph** | ❌ Not supported | ✅ Full support |
| **Mobile performance** | Good (with discipline) | Better (SRP Batcher, render passes) |

Always confirm which pipeline the project uses BEFORE writing any shader.

## 2. URP Shader Standards

### Shader Graph Guidelines
- Keep node graphs **flat** — avoid deeply nested sub-graphs (hard to debug)
- Maximum **50 nodes** per shader graph for mobile (complexity budget)
- Always set **Render Face: Front** (back-face culling saves 50% overdraw)
- Use **Alpha Clip** over **Alpha Blend** (alpha blend = no depth write = overdraw hell)
- Disable unused features: no emission, no specular, no clear coat unless needed

### URP Shader Tier System
| Tier | Shader Type | Use Case | Performance |
|---|---|---|---|
| **Ultra-Light** | Unlit | UI elements, skybox, flat color props | Cheapest |
| **Light** | Simple Lit | Most environment props, NPCs | Low cost |
| **Standard** | Lit (PBR) | Hero characters, key focus objects | Medium |
| **Heavy** | Custom (Toon, Dissolve, Outline) | Special effects, stylized looks | Monitor carefully |

### URP Best Practices
- Enable **SRP Batcher** compatibility on all custom shaders
- Use `_BaseMap` (and not `_MainTex`) for SRP Batcher compatibility
- Prefer **half precision** (`half`, `half3`, `half4`) over `float` where visual difference is negligible

## 3. Built-in Render Pipeline Shader Standards

### Surface Shader Guidelines
- Use **Surface Shaders** for physically-based materials (auto-generates lighting passes)
- Use **Unlit shaders** (`Tags { "RenderType"="Opaque" }` + manual vertex/fragment) for maximum control
- Always specify `#pragma target 3.0` minimum for mobile compatibility

### Built-in Shader Tier System
| Tier | Shader | Use Case |
|---|---|---|
| **Ultra-Light** | `Mobile/Unlit` or custom Unlit | UI, particles, flat objects |
| **Light** | `Mobile/Diffuse` or `Mobile/Bumped Diffuse` | Most props, environment |
| **Standard** | `Standard` (metallic or specular) | Hero models, key items |
| **Custom** | Hand-written CG/HLSL | Toon, dissolve, outline, water |

### Built-in Best Practices
- Avoid `Standard` shader for bulk objects — use `Mobile/` variants
- Dynamic batching works better with Built-in (vs URP SRP Batcher)
- Keep pass count to **1 pass** for mobile (multi-pass = multiple draw calls)
- Use `noforwardadd` pragma to prevent additional per-pixel light passes

### CG/HLSL Coding Rules (Built-in)
```hlsl
// Include guard
#pragma surface surf Lambert noforwardadd noshadow
// Use Lambert instead of Standard for cheaper lighting
// noforwardadd = single light pass only
// noshadow = no shadow receiving (saves a pass)
```

## 4. Texture Channel Packing (Both Pipelines)
Pack multiple data maps into RGB+A channels of a single texture to reduce sampling:

| Channel | Data | Notes |
|---|---|---|
| **R** | Metallic | 0 = dielectric, 1 = metal |
| **G** | Ambient Occlusion | Cavity darkness |
| **B** | Detail Mask or Emission | Context-dependent |
| **A** | Smoothness | Inverse roughness |

This reduces texture samples from 4 separate textures to 1, cutting bandwidth by 75%.

## 5. Fill Rate & Overdraw Management
- **Overdraw budget**: Maximum 2.5× — meaning the average pixel is drawn 2.5 times
- Draw opaque objects **front-to-back** (Unity handles this automatically for opaque queue)
- Draw transparent objects **back-to-front** (expensive — minimize transparent surfaces)
- Particle systems: use **soft particles** sparingly, reduce particle count over billboard size
- Use Unity's **Overdraw** scene view mode to visualize hot spots

## 6. Universal Shader Checklist (Before Shipping)
- [ ] Confirmed render pipeline (URP or Built-in) before writing
- [ ] Mobile-appropriate tier selected (not using Standard shader on 500 props)
- [ ] Back-face culling enabled
- [ ] Alpha Clip preferred over Alpha Blend where possible
- [ ] Texture channel packing applied (Metallic/AO/Smoothness in one map)
- [ ] Half precision used where possible (URP) / Mobile shader variants used (Built-in)
- [ ] Overdraw checked in scene view
- [ ] Shader compiles without warnings on target platform (Android GLES3 / Vulkan)
