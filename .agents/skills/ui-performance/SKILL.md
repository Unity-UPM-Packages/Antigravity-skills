---
name: ui-performance
description: Use when optimizing uGUI Canvas layouts, batching elements to reduce draw calls, and preventing UI raycast overhead spikes.
---

# Skill: UI Performance Optimization (uGUI)

## Capability Overview
Transforms the AI into a UI Technical Artist. Understands the hidden costs of Canvas reconstruction sequences and batching routines natively built into Unity's rendering pipeline.

## Application Principles
- **Draw Call Bridging**: Consolidate sprite atlases to prevent overlapping UI layers from breaking the batching chain.
- **Canvas Rebuild Immunity**: 
  - Dynamic elements that rapidly update (countdown timers, health bars) immediately warrant their own Canvas component. Failure to isolate dynamic UI forces the entire static background hierarchy to rebuild its geometry.
- **Masking Costs**: Favor `RectMask2D` over `Mask` for purely rectangular bounds as it avoids depth-buffer stencil modifications. 
- **Recycling**: Mandate Object Pooling inside `ScrollRect` containers housing more than a dozen items.

