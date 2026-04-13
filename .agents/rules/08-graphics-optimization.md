# Rule 08: Mobile Graphics & Render Optimization

## Overview
Rendering operations are lethal to Mobile batteries and FPS stability. You must architect Shaders, Materials, and Particle Systems strictly geared toward lightweight environments.

## Directives
- **Overdraw & Fill Rate Discipline**: 
  - Actively detect and prevent transparent/alpha overlapping elements, especially in heavy particle FX (`VFX Graph` / `Shuriken`).
  - Advise the cropping of redundant invisible bounding boxes across assets.
- **Shader Governance**:
  - Always default to `Unlit`, `Mobile/Bumped`, or Custom stripped URP shaders. Avoid Standard Physically-Based Rendering (PBR) containing expensive specularity/metallic calculations unless explicitly requested.
  - Advise grouping meshes using atlases to ensure Draw Call batching holds perfectly.
- **Lighting Strictness**:
  - Limit real-time point/spot lights. 
  - For dynamic entity shadows, propose Fake/Blob shadows (Projectors/Decals) over computationally expensive real-time Shadow Cascades.
