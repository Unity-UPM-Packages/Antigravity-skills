---
name: gd-monetization-design
description: "[Game Designer] Use when designing IAP packages, ad placements, Battle Pass structure, pricing strategy, or store UI flow. Applies to any revenue-generating feature design."
---

# Skill: Monetization Feature Design

## When to use this skill
- User asks to design a shop, IAP bundles, or ad placement strategy
- User mentions Battle Pass, VIP subscription, or starter pack
- User wants to optimize ARPDAU or conversion rates

## Decision Tree

| Request | Action |
|---|---|
| "Design a Battle Pass" | Define free/premium tiers → 30–60 day timeline → set XP curve |
| "Design IAP packages" | Apply price anchoring → define value multipliers → create urgency |
| "Place ads" | Map ad slots to natural break points → set frequency caps |
| "Design a starter pack" | Calculate 5x–10x value ratio → one-time purchase → early game trigger |

## Step-by-Step Execution

### Step 1 — IAP Package Template

```
PACKAGE: [Name]
PRICE: [USD]
CONTENTS: [Itemized list with quantities]
VALUE RATIO: [Compared to normal pricing — e.g., "5x value"]
PURCHASE LIMIT: [One-time / Unlimited / Daily refresh]
TRIGGER: [When does the offer appear? Level 3? After first boss?]
URGENCY: [Timer? Limited stock? Seasonal?]
```

### Step 2 — Battle Pass Structure

| Tier | Free Reward | Premium Reward | XP Required |
|---|---|---|---|
| 1 | 100 Gold | Rare Skin Fragment | 0 |
| 2 | 5 Energy | 200 Gems | 500 |
| ... | ... | ... | ... |
| 30 | Nameplate | Legendary Character | 15,000 |

Design rules:
- Free tier must never feel empty (1 reward per tier minimum)
- Premium "hero reward" at tier 25–30 (visible from start as motivation)
- Daily quest XP should complete pass in 80% of season duration (not 100% — creates FOMO)

### Step 3 — Ad Placement Matrix

| Game Moment | Ad Type | Player Mood | Acceptable? |
|---|---|---|---|
| Level Complete (Win) | Rewarded Video (2x coins) | Happy | ✅ Best slot |
| Level Failed | Rewarded Video (Continue) | Frustrated | ✅ If optional |
| Return to Menu | Interstitial | Neutral | ✅ Acceptable |
| Mid-gameplay | Any | Engaged | ❌ NEVER |
| After purchase | Any | Invested | ❌ NEVER |
| Tutorial | Any | Learning | ❌ NEVER |

## Best Practices
- Starter Pack should appear at Level 3–5 (after player understands value of currency)
- Never show more than 2 popup offers per session
- A/B test pricing: $0.99 vs $1.99 vs $2.99 for starter pack
- Battle Pass should be priced at the "impulse buy" threshold ($4.99–$9.99)
