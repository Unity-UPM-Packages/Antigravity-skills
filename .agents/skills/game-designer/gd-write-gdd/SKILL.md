---
name: gd-write-gdd
description: "[Game Designer] Use when writing, expanding, or reviewing a Game Design Document. Activates structured GDD template and player-centric design framework."
---

# Skill: Write Game Design Document (GDD)

## When to use this skill
- User asks to write, draft, or expand a GDD or feature spec
- User describes a mechanic and needs it formalized into a document
- User wants to review an existing design for gaps or inconsistencies

## Decision Tree

| Request Type | Agent Behavior |
|---|---|
| "Write a GDD for X" | Generate full structured GDD using template below |
| "Add a feature to the GDD" | Identify the correct GDD section → insert with scope tag |
| "Review this design" | Check against 5 pillars: Clarity, Measurability, Feasibility, Fun, Scope |
| "Turn this into a tech spec" | Extract mechanic rules → format as Developer-ready implementation notes |

## Step-by-Step Execution

### Step 1 — Fill the Design Brief (ask if missing)
Before writing, confirm:
1. **Genre & Platform**: Mobile? Casual? Midcore?
2. **Core Fantasy**: What does the player *feel*? (Power, exploration, strategy...)
3. **Session Length Target**: 2min casual loop? 20min deep session?

### Step 2 — Generate GDD using Standard Template

```markdown
# [Feature Name] — Game Design Document

## 1. Overview
- **One-line pitch**: [What is it in plain language?]
- **Player Fantasy**: [What emotion does this create?]
- **Priority**: [MUST HAVE / NICE TO HAVE / POST-LAUNCH]

## 2. Core Loop
> Action → Feedback → Reward → Repeat
- **Player Action**: [What does the player DO?]
- **Immediate Feedback**: [What happens visually/audibly right away?]
- **Reward**: [What does the player GAIN?]
- **Loop Driver**: [Why does the player repeat this?]

## 3. Mechanics Specification
[Detail each mechanic with exact rules, numbers, and edge cases]

## 4. Balance Targets
| Parameter | Value | Rationale |
|---|---|---|
| [Stat name] | [Number] | [Why this number] |

## 5. UX Flow
[Describe the screen-by-screen or moment-by-moment player experience]

## 6. Open Questions
- [ ] [Unresolved design decisions that need answers before implementation]

## 7. Developer Notes
[Technical constraints or suggestions for the Developer role]
```

### Step 3 — Flag and Tag
- Mark all scope items: `[MUST HAVE]`, `[NICE TO HAVE]`, `[POST-LAUNCH]`
- Mark unresolved decisions: `[TBD]`
- Mark items ready for dev handoff: `[READY FOR DEV]`

## Best Practices
- Keep Section 3 (Mechanics) precise enough that a developer needs zero follow-up questions
- Never design a mechanic without a corresponding balance number in Section 4
- A GDD is a living document — always note the version and date at the top
