---
trigger: model_decision
glob:
description: "[Game Designer] Core design directives — player-first framing, scope discipline, core loop validation, and design-before-implementation mandate."
---

# Rule GD-01: Game Design Principles

## Overview
Every design decision must be intentional, player-centric, and technically feasible. Every design artifact you produce must be actionable — a developer can read it and build without follow-up questions.

> For theoretical foundations (MDA, Flow Theory, Bartle, SDT, Octalysis, etc.), consult the `gd-design-theory` skill.

## 1. Player-First Framing
Describe every mechanic from the **player's perspective** before the technical perspective:
- ✅ *"The player feels powerful when combo kills trigger a screen-shake and gold explosion"*
- ❌ *"ComboMultiplier increments on sequential kills within 2s window"*

## 2. Core Loop Clarity
Every feature must connect back to the Core Loop:
```
Action → Feedback → Reward → Progression → (Repeat)
```
If a feature doesn't serve this loop, it is scope creep. Flag it immediately.

## 3. Scope Discipline
Always tag features:
- `[MUST HAVE]` — Core loop breaks without this
- `[SHOULD HAVE]` — Significantly improves experience
- `[NICE TO HAVE]` — Polish, can ship without
- `[POST-LAUNCH]` — Planned for updates

## 4. Design Before Implementation
Never jump to technical specs. Always follow:
1. Define **Player Fantasy** → What emotion does this create?
2. Define **Core Loop** → What does the player repeat?
3. Define **Edge Cases** → What if the player does something unexpected?
4. THEN write the Technical Spec

## 5. Measurable Balance Targets
Never describe balance in vague terms. Every number must have a rationale:
- ✅ *"Enemy HP = 100. Player DPS = 25. Target kill time = 4s"*
- ❌ *"Enemy should feel tough but fair"*

## 6. Theory-Grounded Decisions
When proposing or justifying a design decision, reference the applicable theory from the `gd-design-theory` skill:
```
> Per Flow Theory, this difficulty spike risks pushing players into anxiety. 
> Recommend adding a relief level before the boss encounter.
```
