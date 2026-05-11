---
name: gd-progression-design
description: "[Game Designer] Use when designing XP curves, level-up systems, unlock trees, skill trees, content gating, or prestige/rebirth mechanics."
---

# Skill: Progression System Design

## When to use this skill
- User asks to design a leveling system, skill tree, or unlock sequence
- User mentions XP curves, content gating, or prestige mechanics
- User says "progression feels flat" or "nothing to work toward"

## Decision Tree

| Request | Action |
|---|---|
| "Design an XP system" | Define curve shape → generate level table → set unlock rewards |
| "Design a skill tree" | Map branches → define prerequisites → balance opportunity cost |
| "Content gating" | Identify gate type (level/resource/story) → set unlock conditions |
| "Prestige/Rebirth" | Define reset scope → set prestige bonuses → design motivation loop |

## Step-by-Step Execution

### Step 1 — Progression Pillars
Define the axes of progression:

| Axis | Example | Purpose |
|---|---|---|
| **Power** | Level, Stats, Equipment | Player gets stronger |
| **Content** | New levels, modes, characters | Player sees new things |
| **Mastery** | Skill combos, speedrun times | Player gets better personally |
| **Social** | Rank, guild level, titles | Player gains status |

### Step 2 — XP Curve Generation
Select curve type based on design intent:

- **Linear**: `XP(level) = Base + (Level × Step)` — Predictable, good for tutorials
- **Exponential**: `XP(level) = Base × (Multiplier ^ Level)` — Mid-game engagement
- **Polynomial**: `XP(level) = Base × Level²` — Smooth acceleration
- **S-Curve**: Fast early → slow mid → fast late — Best for long-term retention

Always output a **level table**:

| Level | XP Required | Cumulative XP | Unlock | Estimated Time |
|---|---|---|---|---|
| 1 | 0 | 0 | Tutorial | 0 min |
| 2 | 100 | 100 | Feature A | 5 min |
| ... | ... | ... | ... | ... |

### Step 3 — Content Gating Strategy
Map all features to unlock conditions:
- **Hard Gate**: Player CANNOT access until condition met (level requirement)
- **Soft Gate**: Player CAN access but it's difficult (recommended level)
- **Discovery Gate**: Player doesn't know it exists until triggered (hidden content)

## Best Practices
- First 5 levels should take < 10 minutes total (instant gratification)
- Every level-up must grant a VISIBLE reward (not just a number change)
- Prestige systems work only if the "reset" feels like a meaningful choice, not a punishment
- Skill trees should have NO "obviously wrong" paths — every branch viable
