---
trigger: model_decision
glob:
description: Game Designer balance framework — structured approach to tuning numbers, progression curves, and economy design.
---

# Rule GD-02: Balance & Economy Framework

## Overview
Game balance is not guesswork. Apply structured frameworks to produce numbers grounded in design intent, then validate against target player experience.

## Directives

### 1. The Baseline Formula
Establish a baseline before tuning anything:
```
Time-to-Kill (TTK)  = Target HP / Player DPS
Time-to-Die (TTD)   = Player HP / Enemy DPS
Balance Ratio       = TTK / TTD  (target range: 0.8x – 1.5x for fair combat)
```

### 2. Progression Curve Shape
Match the curve shape to the intended experience:
- **Linear**: Predictable, safe. Good for tutorial sections.
- **Exponential**: Accelerating reward. Good for mid-game engagement spikes.
- **Logarithmic**: Diminishing returns. Good for late-game to prevent runaway power.

Always state which curve you are using and why.

### 3. Economy Pillars
For any in-game economy (currency, XP, resources), define all 4 pillars before designing values:
1. **Sources**: Where does the resource come from? (Battle drops, daily rewards, shop)
2. **Sinks**: Where is it spent? (Upgrades, cosmetics, energy refills)
3. **Exchange Rate**: Can it convert to/from other currencies? At what ratio?
4. **Inflation Control**: What caps or decay mechanics prevent infinite hoarding?

### 4. Tuning Velocity
When presenting balance numbers, always include a **tuning handle** — a single variable a developer can expose to the Inspector to adjust during playtesting, without touching the formula logic itself.
