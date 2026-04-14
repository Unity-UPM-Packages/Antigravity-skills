---
name: 2d-elaborate-spec
description: "[2D Artist & UI/UX] Use when receiving a rough spec or art brief from Game Designer and needs to expand it into a production-ready design specification with exact measurements, states, and responsive behavior."
---

# Skill: Elaborate GD Spec into Full Design Spec

## When to use this skill
- GD role has produced a cross-team spec via `gd-cross-team-spec`
- User hands over a rough description and needs it fleshed out
- A feature doc exists but lacks visual/UX detail

## Decision Tree

| GD Spec Contains | Agent Action |
|---|---|
| Screen name + rough element list | Add: exact layout, spacing, component types, all states |
| Wireframe description only | Add: visual treatment (colors, typography tokens), interaction details |
| Art brief only | Add: exact dimensions, export format, 9-slice borders, atlas assignment |
| Technical spec only | Add: visual representation, user-facing flow, error states |

## Step-by-Step Execution

### Step 1 — Parse the GD Spec
Read the incoming spec and identify what's MISSING:

| Required for production | Present? |
|---|---|
| Screen layout with positioning | ☐ |
| Element dimensions (dp/px) | ☐ |
| All interaction states (default, pressed, disabled, loading, error, empty) | ☐ |
| Color references (using palette tokens) | ☐ |
| Typography references (using type scale) | ☐ |
| Spacing values (using grid tokens) | ☐ |
| Responsive behavior notes (16:9 vs 21:9 vs 4:3) | ☐ |
| Animation/transition specs | ☐ |
| Accessibility check (contrast, touch target) | ☐ |

### Step 2 — Fill the Gaps
For every unchecked item, generate the specification:

**Layout elaboration:**
```
Element: [Name]
  Position: [Top-left / Center / Bottom-right / etc.]
  Size: [Width × Height in dp]
  Margin: [top, right, bottom, left in dp — using grid tokens]
  Padding: [internal padding in dp]
```

**State elaboration:**
```
Element: [Button Name]
  Default:  bg=#Primary, text=#White, radius=md, shadow=card
  Pressed:  bg=#PrimaryVariant, scale=0.95, shadow=none
  Disabled: bg=#Gray400, text=#Gray600, alpha=0.6
  Loading:  bg=#Primary, spinner overlay, text hidden
```

**Responsive elaboration:**
```
Aspect Ratio: 16:9 (standard phone)
  → Grid: 3 columns
Aspect Ratio: 21:9 (tall phone)
  → Grid: 3 columns, extra vertical spacing +8dp
Aspect Ratio: 4:3 (tablet)
  → Grid: 4 columns, margins increase to xl
```

### Step 3 — Output Complete Spec
Merge original GD content + your elaborations into ONE self-contained document:
1. Original GD intent (preserved as-is)
2. Visual specification (your additions)
3. Interaction specification (your additions)
4. Asset list with production requirements
5. Developer notes (layout approach, component reuse)

## Best Practices
- Never modify the GD's design intent — only ADD production detail
- Always cross-reference `2d-02-visual-standards` for color/typography tokens
- Always cross-reference `2d-03-production-standards` for naming and export rules
- Flag anything ambiguous with `[NEEDS GD CLARIFICATION]` instead of guessing
