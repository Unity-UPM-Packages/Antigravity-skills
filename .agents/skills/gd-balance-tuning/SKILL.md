---
name: gd-balance-tuning
description: "[Game Designer] Use when tuning combat stats, progression curves, economy values, or difficulty scaling. Applies structured mathematical balance frameworks."
---

# Skill: Balance Tuning

## When to use this skill
- User provides raw stats and asks if they are balanced
- User asks to design an XP curve, economy, or difficulty ramp
- User describes a "feels too easy / too hard" problem during playtesting

## Decision Tree

| Problem Type | Approach |
|---|---|
| "Is this balanced?" | Apply TTK/TTD formula → assess Balance Ratio → flag outliers |
| "Design an XP curve" | Identify target curve shape → generate level table with rationale |
| "Economy feels broken" | Audit 4 economy pillars → identify missing Sink or Source |
| "Difficulty not scaling" | Map current values → identify flat zone → apply multiplier curve |

## Step-by-Step Execution

### Step 1 — Define the Design Intent
Before touching numbers, confirm:
- What is the **target player feeling** at this point in the game? (Challenged? Powerful? Tense?)
- What is the **target session difficulty** (Easy / Normal / Hard)?

### Step 2 — Apply the Balance Formula

**Combat Balance**:
```
TTK (Time-to-Kill) = Enemy HP / Player DPS
TTD (Time-to-Die)  = Player HP / Enemy DPS
Balance Ratio      = TTK / TTD

Healthy range:
  0.8 – 1.2  → Tense, fair fight
  1.2 – 2.0  → Player has advantage (good for early game)
  > 2.0      → Player is overpowered (boring)
  < 0.8      → Enemy is overwhelming (frustrating)
```

**Progression Curve Generator**:
```
Linear:      Value(level) = Base + (Level × Step)
Exponential: Value(level) = Base × (Multiplier ^ Level)
Logarithmic: Value(level) = Base + (Scale × log(Level + 1))
```

### Step 3 — Present as a Tuning Table
Always output a table with a tuning handle column:

| Level | Enemy HP | Player DPS | TTK | Balance Ratio | Tuning Handle |
|---|---|---|---|---|---|
| 1 | 80 | 20 | 4.0s | 1.0 | `_enemyHpMultiplier` |
| 5 | 180 | 20 | 9.0s | 1.1 | `_enemyHpMultiplier` |

### Step 4 — Flag Outliers
After generating the table, scan for:
- Any level where Balance Ratio < 0.7 → `⚠️ Difficulty spike detected at Level X`
- Any level where Balance Ratio > 2.5 → `⚠️ Trivial zone detected at Level X`

## Best Practices
- Always expose tuning handles as `[SerializeField]` fields (note for Dev role)
- Never finalize numbers without a playtest target — state the assumption explicitly
- Separate "paper balance" (math) from "feel balance" (playtesting feedback)
