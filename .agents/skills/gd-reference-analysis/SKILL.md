---
name: gd-reference-analysis
description: "[Game Designer] Use when analyzing a reference game, screenshot, competitor product, or market trend. Performs structured teardown of mechanics, monetization, UX, and art direction."
---

# Skill: Reference & Competitor Analysis

## When to use this skill
- User provides a reference game name or screenshot and asks "analyze this"
- User wants to understand why a competitor's game is successful
- User asks for market positioning or feature comparison

## Decision Tree

| Request | Action |
|---|---|
| "Analyze [Game Name]" | Run full teardown using template below |
| "Compare my game to X" | Feature matrix comparison → identify gaps/advantages |
| "Why is this game successful?" | Reverse-engineer core loop + monetization + retention hooks |
| "Here's a screenshot, analyze it" | UI/UX teardown → art style analysis → mechanics inference |

## Step-by-Step Execution

### Step 1 — Game Teardown Template

```markdown
# Reference Analysis: [Game Name]

## Basic Info
- **Developer**: [Studio name]
- **Genre**: [Category]
- **Platform**: [iOS / Android / Both]
- **Monetization**: [F2P + Ads / IAP / Premium]
- **Store Rating**: [★ score + download estimate]

## Core Loop Analysis
Action → [What player does] 
Feedback → [What happens immediately]
Reward → [What player receives]
Progression → [How this connects to long-term goals]

## Mechanics Breakdown
| Mechanic | Implementation | Why It Works |
|---|---|---|

## Monetization Teardown
| Revenue Source | Implementation | Aggressiveness (1-5) |
|---|---|---|

## UX & Onboarding
- FTUE Duration: [seconds/minutes]
- Tutorial Style: [Guided / Contextual / None]
- First Reward Timing: [When?]

## Art Direction
- Visual Style: [2D Flat / 2D Illustrated / 3D Toon / 3D Realistic]
- Color Palette: [Dominant colors and mood]
- UI Density: [Minimal / Moderate / Heavy]

## What to Steal (Lessons)
1. [Key insight applicable to our project]
2. [...]

## What to Avoid (Weaknesses)
1. [Pain point or bad design decision]
2. [...]
```

### Step 2 — Feature Comparison Matrix
When comparing multiple games:

| Feature | Our Game | Competitor A | Competitor B |
|---|---|---|---|
| Core Loop | | | |
| Monetization | | | |
| Social Features | | | |
| Content Depth | | | |
| Art Quality | | | |

Mark each cell: ✅ Strong / ⚠️ Average / ❌ Weak / — Not present

## Best Practices
- Never copy blindly — understand WHY a feature works in its context
- A feature that works for Clash Royale may fail in a puzzle game
- Focus on systems and loops, not surface-level aesthetics
- Always conclude with actionable "lessons for our project"
