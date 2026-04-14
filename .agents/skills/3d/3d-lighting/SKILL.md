---
name: 3d-lighting
description: "[3D Artist] Use when setting up scene lighting — baked GI, Light Probes, Reflection Probes, lightmap UVs, and balancing artistic quality with mobile performance."
---

# Skill: Lighting Setup

## When to use this skill
- User needs to set up lighting for a scene
- User asks about baked vs real-time lighting tradeoffs
- User wants to optimize lighting for mobile performance

## Decision Tree

| Scenario | Lighting Strategy |
|---|---|
| Static environment, fixed camera | **Fully Baked** (cheapest) |
| Static environment, moving characters | **Baked GI + Light Probes** for dynamic objects |
| Dynamic time-of-day | **Mixed** (baked indirect + 1 real-time directional) |
| Simple stylized (toon/flat) | **Unlit shaders + ambient color** (zero lighting cost) |

## Step-by-Step Execution

### Step 1 — Lighting Plan

```
SCENE: [Name]
MOOD: [Bright daylight / Warm sunset / Dark dungeon / Neon night]
STRATEGY: [Fully Baked / Baked + Probes / Mixed / Unlit]
REAL-TIME LIGHTS: [Count — mobile budget: 1 directional + 0–2 point max]
SHADOWS: [Baked only / 1 real-time directional / Blob shadows]
```

### Step 2 — Light Setup

| Light | Type | Mode | Shadows | Usage |
|---|---|---|---|---|
| Main Sun | Directional | Mixed or Baked | Real-time (cascades=1) or Baked | Primary scene light |
| Fill Light | Directional | Baked | None | Soften harsh shadows |
| Ambient | Skybox or Flat Color | — | — | Fill dark areas |
| Point Lights | Point | Baked | None | Torches, lamps, accents |
| Spot Lights | Spot | Baked | None | Focused highlights |

### Step 3 — Light Probe Placement
For dynamic objects (characters, enemies) moving through baked lighting:

```
Placement rules:
- Grid of probes covering playable area
- Denser near lighting transitions (shadow edges, colored zones)
- Sparser in uniformly lit areas
- At least 2 vertical layers (ground level + head height)
- No probes inside solid geometry
```

### Step 4 — Reflection Probe Setup

| Probe Type | Usage | Update Mode |
|---|---|---|
| **Baked** | Static environments, indoor | On Generate Lighting |
| **Real-time** | Moving reflective objects | Via Script (1/frame max) |
| **Custom** | Cubemap from art | Manual assignment |

Mobile budget: **1–3 baked reflection probes** per scene. Real-time probes are expensive.

### Step 5 — Lightmap Settings (Mobile)

```
Lightmapper: Progressive GPU (fastest bake)
Lightmap Resolution: 10–20 texels/unit (lower = faster, smaller maps)
Lightmap Size: 1024×1024 (mobile) — 2048 max for detailed scenes
Compress Lightmaps: ✅
Ambient Occlusion: ✅ (baked in — free at runtime)
Directional Mode: Non-Directional (cheaper) or Directional (better quality)
```

**Lightmap UV2 Requirements:**
- Non-overlapping
- No stretch
- Generated automatically in Unity OR manually in Blender (UV Channel 1)

### Step 6 — Mobile Shadow Alternatives

| Technique | Quality | Performance | When |
|---|---|---|---|
| **Real-time Shadow Map** | Best | Expensive (1 light only) | Hero character, key moments |
| **Baked Shadows** | Good (static) | Free at runtime | Static scene elements |
| **Blob Shadow (Projector)** | Acceptable | Very cheap | NPCs, enemies, dynamic objects |
| **Fake Shadow (Decal/Quad)** | Basic | Cheapest | Mobile-first indie games |

## Best Practices
- **One real-time shadow-casting light MAX** on mobile (the directional sun)
- Real-time shadow cascades: **1 cascade** on mobile (2 max on midrange)
- All other lights: baked or blob shadow fallback
- Test lighting on a dark device screen (OLED vs LCD differ dramatically)
- Mark all static geometry as **Contribute GI: ✅** and **Receive GI: ✅**
