---
name: 2d-art-direction
description: "[2D Artist & UI/UX] Use when creating style guides, color palettes, design token systems, UI kits, or visual identity documents for a game project."
---

# Skill: Art Direction & Style Guide

## When to use this skill
- User asks to define the visual identity of a project
- User needs a color palette, typography selection, or mood reference
- User wants to create a UI Kit or component library
- GD has provided an art brief that needs visual expansion

## Step-by-Step Execution

### Step 1 — Visual Identity Brief
Gather or confirm from GD spec:

```
PROJECT: [Game Name]
GENRE: [Genre]
MOOD: [Playful / Dark / Minimalist / Vibrant / Epic / Cozy]
TARGET AUDIENCE: [Casual / Midcore / Hardcore]
ART STYLE REFERENCE: [Reference games or visual examples]
```

### Step 2 — Color Palette Definition

```markdown
## Color Palette

### Primary Palette
| Role | Hex | Usage |
|---|---|---|
| Primary | #[hex] | Main CTA buttons, key highlights |
| Primary Variant | #[hex] | Pressed states, hover |
| Secondary | #[hex] | Secondary actions, accents |

### Neutral Palette
| Role | Hex | Usage |
|---|---|---|
| Background | #[hex] | Main screen background |
| Surface | #[hex] | Cards, panels, modals |
| Divider | #[hex] | Thin separator lines |

### Semantic Palette
| Role | Hex | Usage |
|---|---|---|
| Success | #[hex] | Positive feedback, rewards |
| Warning | #[hex] | Caution states |
| Error | #[hex] | Destructive actions, failures |
| Info | #[hex] | Informational highlights |

### Rarity Colors (for item/character systems)
| Rarity | Hex | Glow/Border Treatment |
|---|---|---|
| Common | #[hex] | No glow |
| Uncommon | #[hex] | Subtle border |
| Rare | #[hex] | Soft glow |
| Epic | #[hex] | Bright glow |
| Legendary | #[hex] | Animated shimmer |
```

### Step 3 — Typography Selection

```markdown
## Typography

| Role | Font Family | Weight | Size | Letter Spacing |
|---|---|---|---|---|
| Display | [Font] | Bold | 28–32sp | -0.5% |
| Heading | [Font] | SemiBold | 20–24sp | 0 |
| Body | [Font] | Regular | 14–16sp | 0 |
| Caption | [Font] | Regular | 12sp | +0.5% |
| Button | [Font] | Medium | 14sp | +1% |
| Number | [Monospace variant] | Bold | varies | Tabular |
```

### Step 4 — Design Tokens
Export as a structured token system for dev handoff:

```json
{
  "color": {
    "primary": "#1976D2",
    "secondary": "#00897B",
    "background": "#1A1A2E",
    "surface": "#252545",
    "error": "#D32F2F"
  },
  "spacing": {
    "xs": 4, "sm": 8, "md": 16, "lg": 24, "xl": 32
  },
  "radius": {
    "sm": 4, "md": 8, "lg": 16, "pill": 999
  },
  "shadow": {
    "card": "0 2dp 8dp rgba(0,0,0,0.25)",
    "modal": "0 8dp 24dp rgba(0,0,0,0.5)"
  }
}
```

### Step 5 — UI Component Library
Define the visual treatment of core components:

| Component | Variants | States |
|---|---|---|
| Button | Primary, Secondary, Ghost, Icon-only | Default, Pressed, Disabled, Loading |
| Card | Item Card, Character Card, Reward Card | Default, Selected, Locked |
| Dialog | Confirm, Alert, Info | Open, Closing |
| Input | Text, Search, Number | Empty, Focused, Filled, Error |
| Badge | Notification, Rarity, Level | — |
| Progress Bar | HP, XP, Loading | Filling, Full, Depleting |

## Best Practices
- Test color palette in both light AND dark environments
- Always verify contrast ratios against `2d-02-visual-standards`
- Design tokens bridge the gap between Design and Dev — always output them
- Rarity colors should feel premium at higher tiers (glow, shimmer, particle effects)
