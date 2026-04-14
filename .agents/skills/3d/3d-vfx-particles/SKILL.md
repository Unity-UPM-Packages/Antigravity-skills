---
name: 3d-vfx-particles
description: "[3D Artist] Use when creating visual effects using Unity Particle System or VFX Graph — explosions, fire, magic, hit impacts, trails, and mobile-safe effect optimization."
---

# Skill: VFX & Particle Systems

## When to use this skill
- User needs a visual effect (explosion, fire, magic spell, hit impact)
- User asks about Particle System vs VFX Graph
- User wants to optimize existing VFX for mobile

## Decision Tree

| Effect Type | System | Rationale |
|---|---|---|
| Simple effects (dust, sparks, blood) | **Particle System (Shuriken)** | Lightweight, CPU-based, universal |
| Complex effects (fluid, thousands of particles) | **VFX Graph** | GPU-based, requires compute shader |
| Mobile target | **Particle System** preferred | VFX Graph requires GPU compute — not all mobile GPUs support it |

## Step-by-Step Execution

### Step 1 — VFX Brief

```
EFFECT: [Name]
TYPE: [Explosion / Fire / Magic / Hit / Trail / Ambient / UI]
TRIGGER: [On spawn / On impact / Continuous / Event-driven]
DURATION: [Seconds — or Looping]
PARTICLE COUNT: [Peak visible count]
PERFORMANCE TIER: [Low-end safe / Mid-range / High-end only]
```

### Step 2 — Particle System Setup Template

```markdown
## Effect: [Name]

### Emission
- Rate: [N particles/sec] or Burst: [N at time 0]
- Max Particles: [cap — e.g., 30]

### Particle Properties
| Property | Value | Over Lifetime |
|---|---|---|
| Start Size | [0.1 – 2.0] | [Shrink curve / Grow then shrink] |
| Start Speed | [0 – 10] | [Decelerate] |
| Start Color | [Hex + alpha] | [Fade out alpha] |
| Start Lifetime | [0.3 – 2.0s] | — |
| Gravity | [0 – 1] | — |

### Renderer
- Render Mode: [Billboard / Stretched Billboard / Mesh]
- Material: [Additive / Alpha Blended / Custom]
- Sort Mode: [By Distance / Oldest First]

### Sub-Emitters (if needed)
| Trigger | Sub-Effect |
|---|---|
| On Death | Smoke puff |
| On Collision | Spark burst |
```

### Step 3 — Mobile Optimization Rules

| Constraint | Limit | Impact |
|---|---|---|
| Max particles per effect | 30 – 50 | CPU overhead |
| Max concurrent effects on screen | 5 – 10 | Combined overdraw |
| Max overdraw layers | 3 | Fill rate killer |
| Texture size per effect | 128 or 256 | Memory |
| Shader | Additive or Alpha Clip | Avoid alpha blend stacking |

### Optimization Techniques
- Use **flipbook** (spritesheet animation) instead of many layered particles
- Use **GPU Instancing** on particle materials where supported
- Use **mesh particles** instead of billboards for 3D-looking effects (fewer particles needed)
- Disable **collision** module unless gameplay-critical
- Set **Culling Mode: Automatic** — particles off-screen stop simulating

## Common Effect Recipes

| Effect | Key Trick |
|---|---|
| **Explosion** | Burst emission (50 at t=0) → rapid size increase → alpha fade |
| **Fire** | Looping emission → upward velocity → noise turbulence → warm color gradient |
| **Magic circle** | Mesh particle (flat plane) → rotation over lifetime → emission glow |
| **Hit impact** | Short burst → stretch billboard → rapid shrink |
| **Trail** | Trail module on moving object → width curve shrinks over distance |
| **Dust/debris** | Low emission → gravity → random size → short lifetime |

## Best Practices
- Every effect must have a defined **end** (no infinite particle leaks)
- Pool particle systems — never Instantiate/Destroy at runtime
- Test on LOW-END device first — scale UP quality, never down
- Use the **Frame Debugger** to count overdraw per effect
