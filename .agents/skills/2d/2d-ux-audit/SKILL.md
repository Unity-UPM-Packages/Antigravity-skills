---
name: 2d-ux-audit
description: "[2D Artist & UI/UX] Use when reviewing or auditing an existing UI design for usability issues, accessibility violations, visual inconsistencies, or deviation from project standards."
---

# Skill: UX Audit & Review

## When to use this skill
- User shares a screenshot or description of an existing UI and asks for feedback
- User suspects a screen has usability problems
- User wants a pre-launch UI quality review

## Step-by-Step Execution

### Step 1 — Run the Audit Checklist
Score each category 1–5 (1=Critical issues, 5=Excellent):

```markdown
## UX Audit: [Screen Name]

### Category Scores
| Category | Score | Notes |
|---|---|---|
| **Clarity** — Is the purpose of this screen immediately obvious? | /5 | |
| **Hierarchy** — Is the primary action the most prominent element? | /5 | |
| **Touch Targets** — Are all interactive elements ≥ 44×44dp? | /5 | |
| **Feedback** — Do all interactive elements have visible pressed/disabled states? | /5 | |
| **Navigation** — Can the user get back? Do they know where they are? | /5 | |
| **Consistency** — Do colors, fonts, spacing match the style guide? | /5 | |
| **Accessibility** — Do contrast ratios meet WCAG minimum (4.5:1 text, 3:1 icons)? | /5 | |
| **Empty/Error States** — Are loading, empty, and error states designed? | /5 | |
| **Information Density** — Is the screen too crowded or too sparse? | /5 | |
| **Responsive** — Will this work on 16:9, 21:9, and 4:3? | /5 | |

**Overall Score**: X/50
```

### Step 2 — Issue Log
For each problem found, create an actionable issue:

```markdown
### Issue #[N]
- **Severity**: 🔴 Critical / 🟡 Warning / 🟢 Suggestion
- **Category**: [from checklist above]
- **Element**: [specific UI element]
- **Problem**: [what's wrong]
- **Recommendation**: [how to fix it]
- **Reference**: [which rule/standard is violated — e.g., 2d-01-ux-principles §2]
```

### Step 3 — Priority Matrix

| Issue | Severity | Impact on UX | Fix Effort | Priority |
|---|---|---|---|---|
| #1 | 🔴 | High | Low | **FIX NOW** |
| #2 | 🟡 | Medium | Medium | Fix before launch |
| #3 | 🟢 | Low | Low | Nice improvement |

## Common Issues Checklist (Quick Scan)
- [ ] Text is readable on all backgrounds (check contrast)
- [ ] Buttons look tappable (not flat/invisible)
- [ ] Currency/resource display is always visible during purchase flows
- [ ] No text truncation on any supported language/device
- [ ] Scroll indicators are visible when content overflows
- [ ] Modal dialogs can be dismissed (X or tap-outside)
- [ ] Destructive actions require confirmation dialog
- [ ] Loading states prevent double-tap submissions

## Best Practices
- Be specific: "Button X is 32×32dp, should be 44×44dp" not "buttons are small"
- Always reference the violated standard by name and section
- Prioritize usability over aesthetics — a beautiful screen that confuses users is a failure
