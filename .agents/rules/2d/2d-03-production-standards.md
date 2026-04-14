---
trigger: always_on
glob:
description: "[2D Artist & UI/UX] Production standards — file naming, component organization, export settings, and developer handoff checklist."
---

# Rule 2D-03: Production & Handoff Standards

## Overview
Clean production pipelines save hours of debugging. Every asset, layer, and component must follow strict naming and organization rules to ensure seamless handoff to developers.

## 1. File & Layer Naming Convention

### Pattern
```
[category]/[screen]-[element]-[variant]-[state]
```

### Examples
| Asset | Name |
|---|---|
| Shop screen background | `bg/shop-main` |
| Buy button default | `btn/shop-buy-default` |
| Buy button pressed | `btn/shop-buy-pressed` |
| Buy button disabled | `btn/shop-buy-disabled` |
| Gold coin icon | `ico/currency-gold` |
| Player avatar frame | `frame/avatar-player-default` |

### Rules
- All lowercase, hyphen-separated (no spaces, no underscores, no camelCase)
- States always as suffix: `-default`, `-pressed`, `-disabled`, `-hover`, `-selected`
- Variants as suffix before state: `-primary`, `-secondary`, `-small`, `-large`

## 2. Component Organization

### Atomic Design Structure
Organize UI components in layers of complexity:

| Level | Definition | Example |
|---|---|---|
| **Atoms** | Single indivisible elements | Button, Icon, Text label, Avatar |
| **Molecules** | Groups of atoms | Search bar (icon + input + button) |
| **Organisms** | Groups of molecules | Header bar (logo + search + profile) |
| **Templates** | Page-level layouts | Shop screen layout (header + grid + footer) |

### Component Rules
- Every reusable element must be a **component** (not a one-off shape)
- Components must have **all states defined** (default, pressed, disabled, etc.)
- Components use **auto-layout** or equivalent for responsive behavior

## 3. Export Settings

### For Unity (uGUI)
| Asset Type | Format | Sizing | Compression |
|---|---|---|---|
| UI Sprites | PNG-24 (alpha) | @1x at target resolution | ASTC in Unity |
| Icons | PNG-24 (alpha) | Export at 2× then downscale | ASTC 4×4 |
| Backgrounds | JPG (no alpha) or PNG | Match device resolution | ASTC 6×6 |
| 9-Slice elements | PNG-24 + slice metadata | Minimum stretchable size | ASTC 4×4 |

### 9-Slice Export Checklist
- [ ] Define slice borders (left, right, top, bottom in pixels)
- [ ] Document borders in the asset spec sheet
- [ ] Test at 0.5×, 1×, and 2× scale to verify stretching

## 4. Developer Handoff Checklist
Before declaring a screen "ready for dev":

- [ ] All elements are named following convention (Section 1)
- [ ] All interactive elements have ALL states designed (default, pressed, disabled, loading)
- [ ] Spacing values use the grid system tokens from `2d-02-visual-standards`
- [ ] Colors reference the defined color roles (Primary, Secondary, etc.)
- [ ] Font sizes map to the type scale (H1, Body, Caption, etc.)
- [ ] Touch targets meet minimum 44×44dp requirement
- [ ] 9-slice borders are documented for all stretchable elements
- [ ] Export assets are organized in folders matching Unity's project structure
- [ ] Animation specifications are attached (duration, easing, trigger)
- [ ] Responsive behavior is annotated (what happens on 16:9 vs 21:9 vs 4:3?)
