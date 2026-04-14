---
name: gd-level-design
description: "[Game Designer] Use when designing level layouts, difficulty curves, session pacing, boss encounters, or puzzle sequences. Covers structure, flow, and reward placement."
---

# Skill: Level Design & Difficulty Pacing

## When to use this skill
- User asks to design a level, stage, or mission structure
- User mentions difficulty curve, pacing, or "the game gets boring at level X"
- User wants to design boss encounters or puzzle sequences

## Decision Tree

| Request | Action |
|---|---|
| "Design a level" | Apply Kiss-Start → Teach → Test → Reward structure |
| "Difficulty curve" | Map current curve → identify spikes/plateaus → smooth |
| "Design a boss fight" | 3-phase pattern → telegraph → punish window → reward |
| "Pacing feels off" | Measure intensity peaks/valleys → apply wave pattern |

## Step-by-Step Execution

### Step 1 — Level Structure Template

```
LEVEL: [Number / Name]
ENVIRONMENT: [Visual theme / setting]
OBJECTIVE: [Clear win condition]
NEW MECHANIC INTRODUCED: [What does the player learn here?]
ESTIMATED DURATION: [Session length target]
DIFFICULTY: [1-10 scale relative to previous level]
REWARD: [What the player earns for completion]
```

### Step 2 — Difficulty Curve Design
The ideal curve is NOT a straight line — it's a wave:

```
Intensity
  ▲
  │     ╱╲       ╱╲╲      ╱╲
  │   ╱╱  ╲    ╱╱   ╲╲  ╱╱  ╲╲   ← Boss
  │  ╱     ╲╱╱╱      ╲╱╱     ╲╲
  │╱╱                            ╲
  └──────────────────────────────► Levels
   Tutorial   Mid-game    Endgame
```

**Rules:**
- After every difficulty spike, provide a **rest level** (easier, rewarding)
- Every 5th level should be noticeably easier (relief valve)
- Boss fights are the peaks — always preceded by a tutorial of the boss's mechanic

### Step 3 — Boss Encounter Template

| Phase | Boss Behavior | Player Response | Telegraph |
|---|---|---|---|
| Phase 1 (100%–70% HP) | Simple attack pattern | Learn pattern | Obvious visual cue |
| Phase 2 (70%–30% HP) | Add new mechanic | Adapt strategy | Medium visual cue |
| Phase 3 (30%–0% HP) | Enrage / all mechanics combined | Execute mastery | Subtle cue |

### Step 4 — Session Pacing for Mobile
Mobile sessions must fit into short bursts:

| Session Type | Duration | Content |
|---|---|---|
| **Micro-session** | 1–3 min | 1 level or 1 battle |
| **Standard session** | 5–10 min | 3–5 levels or 1 event run |
| **Deep session** | 15–30 min | Story chapter or competitive mode |

Design levels so players can **stop at any point** without losing progress.

## Best Practices
- The first level should be impossible to fail (build confidence)
- Never introduce 2 new mechanics in the same level
- Reward placement: put the most exciting reward RIGHT AFTER the hardest section
- If a level has > 30% failure rate, the design is wrong — not the player
