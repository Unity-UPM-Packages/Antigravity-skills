---
name: gd-player-lifecycle
description: "[Game Designer] Use when designing onboarding flows, tutorials, retention mechanics, daily login systems, live events, or social engagement hooks. Covers the full player journey from install to long-term retention."
---

# Skill: Player Lifecycle & Retention Design

## When to use this skill
- User asks about tutorial design, onboarding, or FTUE (First-Time User Experience)
- User mentions retention metrics (D1, D7, D30), churn, or engagement
- User wants to design daily login, events, or social features

## Decision Tree

| Request | Action |
|---|---|
| "Design a tutorial" | Map FTUE flow → teach by doing → validate with 3-click rule |
| "Improve retention" | Identify lifecycle stage → apply matching hook type |
| "Design daily login" | Create escalating reward calendar → add streak bonuses |
| "Design live events" | Define event loop → set duration → design exclusive rewards |

## Step-by-Step Execution

### Step 1 — Player Lifecycle Mapping

| Phase | Timeline | Goal | Key Mechanic |
|---|---|---|---|
| **Discovery** | Pre-install | Attract & convert | Store page, ads, virality |
| **Onboarding** | Session 1 (0–5 min) | Teach core loop | Interactive tutorial, no text walls |
| **Habit Formation** | Day 1–7 | Build daily habit | Daily rewards, push notifications |
| **Mid-game** | Week 2–4 | Deepen engagement | Unlock new systems, social features |
| **Endgame** | Month 2+ | Sustain & monetize | Prestige, competitive, live events |
| **Churn Risk** | Any time | Recover lapsed players | Re-engagement offers, comeback bonuses |

### Step 2 — FTUE (Tutorial) Design Rules
1. **Show, don't tell** — Player learns by doing, not reading
2. **3-Click Rule** — Player should perform their first meaningful action within 3 taps
3. **Delayed Complexity** — Introduce ONE mechanic per tutorial step
4. **Reward immediately** — First reward within 30 seconds of starting
5. **Skip option** — Always allow experienced players to skip (after core loop is shown)

### Step 3 — Retention Hook Library

| Hook Type | Mechanism | Best For |
|---|---|---|
| **Daily Login Calendar** | Escalating rewards over 7/14/28 days | D1–D7 retention |
| **Streak Bonus** | Consecutive day multiplier (breaks reset) | Habit formation |
| **Daily Quests** | 3–5 achievable tasks → bonus chest | Session depth |
| **Limited-Time Events** | 3–7 day themed content with exclusive rewards | Re-engagement |
| **Push Notifications** | "Your energy is full!" / "Free gift waiting!" | Lapsed player recovery |
| **Social Hooks** | Friends list, guilds, co-op, gifting | Long-term retention |
| **Seasonal Content** | New levels/skins tied to real-world calendar | Anticipation |

### Step 4 — Retention Metrics Target

| Metric | Casual | Midcore | Hardcore |
|---|---|---|---|
| D1 Retention | ≥ 40% | ≥ 35% | ≥ 30% |
| D7 Retention | ≥ 15% | ≥ 12% | ≥ 10% |
| D30 Retention | ≥ 5% | ≥ 4% | ≥ 3% |
| Avg Session Length | 3–5 min | 10–20 min | 20–60 min |
| Sessions/Day | 3–5 | 2–4 | 1–3 |

## Best Practices
- Never punish players for NOT logging in (negative reinforcement destroys goodwill)
- Streak rewards should reset gracefully (1 miss = back to day 1 is too harsh; try −2 days)
- Push notifications: max 2/day, always with clear value ("Free chest ready!")
- Tutorial completion rate is your most critical metric — if < 70% finish, redesign immediately
