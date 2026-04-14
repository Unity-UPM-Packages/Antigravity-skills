---
trigger: always_on
glob:
description: "[2D Artist & UI/UX] Mobile UX constraints — touch targets, accessibility, navigation patterns, feedback states, and information hierarchy."
---

# Rule 2D-01: Mobile UX Principles

## Overview
Every UI screen must be usable, accessible, and intuitive. Players should never feel lost, confused, or frustrated by the interface — only by the game's challenge.

## 1. Touch Target Standards

| Element | Minimum Size | Recommended Size |
|---|---|---|
| Primary action button (Attack, Confirm) | 44×44 dp | 48×48 dp |
| Secondary action (Back, Close) | 36×36 dp | 44×44 dp |
| List items / Grid cells | 44dp height | 48dp height |
| Spacing between tappable elements | 8dp minimum | 12dp recommended |

> **dp** = density-independent pixels. On a 2x display, 44dp = 88 physical pixels.

⚠️ Never place two destructive actions (Delete, Purchase) adjacent without adequate spacing.

## 2. Accessibility (WCAG Mobile)

### Color Contrast
- **Body text**: Contrast ratio ≥ 4.5:1 against background
- **Large text / Icons**: Contrast ratio ≥ 3:1
- **Never rely on color alone** to convey information (colorblind users). Always pair color with icon/shape/text

### Readability
- Minimum font size: 12sp for body text, 10sp absolute minimum for labels
- Line height: 1.4× – 1.6× font size
- Maximum line length: 40–60 characters per line for readability

## 3. Navigation Patterns

### Mobile Game UI Hierarchy
```
Root (Main Menu)
  ├── Play / Core Loop Entry
  ├── Shop / Store
  ├── Inventory / Collection
  ├── Social / Friends
  └── Settings
```

### Navigation Rules
- **Depth limit**: Maximum 3 taps to reach any feature from Root
- **Back button**: Every screen MUST have a clear back/close affordance
- **Current location**: Player must always know WHERE they are (highlighted tab, breadcrumb, title)
- **No dead ends**: Every screen must have an exit path

## 4. UI Feedback States
Every interactive element MUST define all 5 states:

| State | Visual Treatment |
|---|---|
| **Default** | Normal appearance |
| **Pressed** | Scale down 0.95 + darken 10% + sound |
| **Disabled** | Grayscale or 50% alpha, no interaction |
| **Loading** | Spinner overlay or skeleton placeholder |
| **Empty** | Friendly message + suggested action ("No items yet. Visit the shop!") |

## 5. Safe Zones
- Top 44dp: Reserved for system status bar (iOS notch, Android status)
- Bottom 34dp: Reserved for iOS home indicator
- Always wrap gameplay HUD inside a SafeArea container
