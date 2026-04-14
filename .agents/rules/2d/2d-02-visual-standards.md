---
trigger: always_on
glob:
description: "[2D Artist & UI/UX] Visual design standards — color theory, typography hierarchy, spacing systems, and visual consistency enforcement."
---

# Rule 2D-02: Visual Design Standards

## Overview
Visual consistency is the difference between a polished product and a prototype. Every screen, every element, and every state must follow a unified visual language.

## 1. Color System

### Color Roles
Every project must define these color roles before any UI work begins:

| Role | Purpose | Example |
|---|---|---|
| **Primary** | Main brand color, primary actions | Blue (#1976D2) |
| **Secondary** | Supporting actions, accents | Teal (#00897B) |
| **Background** | Surface colors, cards, panels | Dark (#1A1A2E), Light (#F5F5F5) |
| **Surface** | Elevated elements (modals, tooltips) | Slight lift from background |
| **Error** | Destructive actions, warnings | Red (#D32F2F) |
| **Success** | Confirmations, positive feedback | Green (#388E3C) |
| **Text Primary** | Main content text | High contrast on background |
| **Text Secondary** | Labels, captions, hints | 60% opacity of primary |

### Color Usage Rules
- Maximum **3–4 dominant colors** per screen (excluding grayscale)
- Use **saturation variation** rather than entirely different hues for visual harmony
- Dark mode: Never use pure black (#000000) — use near-black (#121212 or #1A1A2E)
- Light mode: Never use pure white (#FFFFFF) for large surfaces — use warm white (#FAFAFA)

## 2. Typography Hierarchy

### Scale Definition
Define a consistent type scale — every text element must map to ONE of these:

| Level | Use Case | Size | Weight |
|---|---|---|---|
| **H1** | Screen titles | 24–28sp | Bold |
| **H2** | Section headers | 20–22sp | SemiBold |
| **H3** | Card titles, dialog headers | 16–18sp | SemiBold |
| **Body** | Main content, descriptions | 14–16sp | Regular |
| **Caption** | Labels, timestamps, hints | 12sp | Regular |
| **Overline** | Category tags, currency labels | 10–11sp | Medium / AllCaps |

### Typography Rules
- Use **maximum 2 font families** per project (1 for headings, 1 for body — or just 1 with weight variations)
- Line spacing: 1.4× for body, 1.2× for headings
- Letter spacing: +0.5% for AllCaps text (improves readability)
- Numeric text (scores, currency): Use **tabular/monospace figures** so digits don't shift during animation

## 3. Spacing & Grid System

### Base Unit
Define a **4dp or 8dp base grid**. All spacing must be a multiple:

| Token | Value (8dp base) | Usage |
|---|---|---|
| `xs` | 4dp | Icon padding, dense lists |
| `sm` | 8dp | Element inner padding |
| `md` | 16dp | Card padding, section gaps |
| `lg` | 24dp | Screen margins |
| `xl` | 32dp | Major section separation |
| `xxl` | 48dp | Hero area breathing room |

### Consistency Rule
- If a button has `16dp` horizontal padding in the Shop, it must have `16dp` in Settings too
- Never use magic numbers (13dp, 17dp, 22dp) — always snap to the grid

## 4. Visual Hierarchy Checklist
For every screen, verify:
- [ ] The most important action is the **largest and most colorful** element
- [ ] Secondary actions are visually **subordinate** (smaller, less saturated, outlined instead of filled)
- [ ] There is clear **visual grouping** (related items are close together, unrelated items have spacers)
- [ ] The eye naturally flows in a **Z-pattern** (top-left → top-right → bottom-left → bottom-right) or **F-pattern** for content-heavy screens
- [ ] **White space** is intentional, not accidental — crowded screens are hostile screens

## 5. Iconography
- Use **a single icon style** throughout: Outlined OR Filled, not mixed
- Icon size: 24×24dp standard, 16×16dp small, 32×32dp large
- Always include a text label alongside icons for non-obvious actions (exceptions: universally understood icons like ✕, ←, ⚙)
- Icon stroke width: consistent across the set (2dp recommended)
