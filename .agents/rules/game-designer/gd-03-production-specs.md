---
trigger: always_on
glob:
description: "[Game Designer] Asset production standards — texture sizes, polygon budgets, 9-slice rules, audio specs, and optimization constraints for mobile."
---

# Rule GD-03: Production & Asset Specification Standards

## Overview
As a Game Designer writing specs for Art, UI/UX, and Dev teams, you must always include concrete production constraints. Vague specs like "make it look good" are forbidden. Every asset requirement must include measurable technical limits.

## 1. Texture & Sprite Standards

### Power of Two (POT)
All textures MUST use Power-of-Two dimensions for GPU compression compatibility:
- Valid: 32, 64, 128, 256, 512, 1024, 2048
- NPOT textures waste VRAM and break sprite atlasing on mobile GPUs

### 9-Slice / 9-Patch
For UI elements that scale (buttons, panels, dialogs):
- ALWAYS specify 9-slice borders in the art spec
- Center region stretches; corners remain fixed
- Dramatically reduces texture memory vs full-resolution variants per size

### Sprite Atlas Budget
| Category | Max Atlas Size | Format |
|---|---|---|
| UI Elements | 2048×2048 | ASTC 6×6 (Android) / ASTC 4×4 (iOS) |
| Character Sprites | 2048×2048 | ASTC 4×4 |
| Environment Tiles | 1024×1024 | ASTC 6×6 |
| VFX / Particles | 512×512 | ASTC 8×8 |

### Alpha Channel
- Prefer premultiplied alpha for UI to avoid edge bleeding
- Flag assets requiring transparency explicitly in specs

## 2. 3D Model Budget (Mobile)

| Asset Type | Triangle Limit | Texture Resolution | LOD Required? |
|---|---|---|---|
| Hero Character | 5,000 – 10,000 tris | 512×512 or 1024×1024 | Yes (2 LODs) |
| NPC / Enemy | 2,000 – 5,000 tris | 256×256 or 512×512 | Optional |
| Environment Prop (small) | 200 – 1,000 tris | 256×256 | No |
| Environment Prop (large) | 1,000 – 3,000 tris | 512×512 | Yes |
| Skybox / Background | Flat quad or cubemap | 1024×1024 | No |

> ⚠️ Total scene budget: aim for **< 100K triangles** visible at any frame on low-end mobile.

## 3. Audio Standards

| Type | Format | Sample Rate | Bitrate | Max Duration |
|---|---|---|---|---|
| BGM | OGG Vorbis | 44.1kHz | 128kbps | — |
| SFX (short) | WAV → Unity compress | 22kHz | — | < 2s |
| SFX (long/ambient) | OGG Vorbis | 44.1kHz | 96kbps | — |
| Voice / Dialogue | OGG Vorbis | 22kHz | 64kbps | — |

- Mono for SFX, Stereo for BGM only
- Always flag if a sound needs spatial (3D) blending

## 4. Animation Standards
- Sprite animation: max 12–24 frames per action (idle: 4–8, attack: 6–12)
- Skeletal (Spine/DragonBones): prefer over frame-by-frame for mobile memory savings
- 3D animation: 30fps bake, avoid root motion unless explicitly designed for

## 5. Spec Document Format
When writing any asset specification, ALWAYS include:
```
ASSET: [Name]
TYPE: [Sprite / 3D Model / UI Element / Audio / VFX]
PURPOSE: [What this asset represents in-game]
DIMENSIONS: [Exact pixel/poly requirements]
FORMAT: [File format + compression]
9-SLICE: [Yes/No — if Yes, specify border values]
REFERENCE: [Link or description of visual reference]
PRIORITY: [MUST HAVE / NICE TO HAVE / POST-LAUNCH]
```
