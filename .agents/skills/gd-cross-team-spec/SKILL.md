---
name: gd-cross-team-spec
description: "[Game Designer] Use when writing specifications for other departments: wireframes for UI/UX designers, art direction for 2D/3D artists, technical specs with algorithms for developers. The master cross-functional communication skill."
---

# Skill: Cross-Team Specification Writer

## When to use this skill
- User asks to write specs for developers, artists, or UI designers
- User says "how do I communicate this design to the team?"
- User needs wireframes, art briefs, or technical algorithm proposals

## Decision Tree

| Audience | Output Type |
|---|---|
| **UI/UX Designer** | Wireframe description + annotation + interaction notes |
| **2D Artist** | Art brief with style reference + color palette + asset list |
| **3D Artist** | Model spec with poly budget + texture size + animation list |
| **Developer** | Technical spec with data structures + algorithm pseudocode + API contract |
| **Sound Designer** | Audio brief with mood + trigger events + duration constraints |

## Step-by-Step Execution

### For UI/UX Designers — Wireframe Spec

```markdown
## Screen: [Screen Name]

### Layout Description
[Describe the screen structure top-to-bottom, left-to-right]

### Element List
| Element | Type | Position | Interaction | Notes |
|---|---|---|---|---|
| [Health Bar] | ProgressBar | Top-left | None (reactive) | Updates via event |
| [Attack Button] | Button | Bottom-right | Tap → trigger attack | Cooldown overlay |

### Navigation Flow
- [Tap X] → goes to [Screen Y]
- [Swipe left] → shows [Panel Z]

### Responsive Notes
- Anchor health bar to top-left with SafeArea padding
- Button size: minimum 44×44 dp touch target (Apple HIG)
- 9-slice panel backgrounds for all dialog boxes

### Interaction States
| Element | Default | Pressed | Disabled | Loading |
|---|---|---|---|---|
| [Button] | Blue | Dark blue + scale 0.95 | Gray + 50% alpha | Spinner overlay |
```

### For 2D Artists — Art Direction Brief

```markdown
## Art Brief: [Asset/Feature Name]

### Visual Style
- **Reference**: [Attach reference images or describe style]
- **Mood**: [Playful / Dark / Minimalist / Vibrant]
- **Color Palette**: [Primary: #hex, Secondary: #hex, Accent: #hex]

### Asset List
| Asset | Type | Dimensions | 9-Slice? | Priority |
|---|---|---|---|---|
| [Main Menu BG] | Sprite | 1080×1920 | No | MUST HAVE |
| [Dialog Panel] | Sprite | 600×400 | Yes (24px border) | MUST HAVE |
| [Coin Icon] | Sprite | 64×64 | No | MUST HAVE |

### Technical Constraints (from gd-03-production-specs)
- All sprites: POT dimensions, ASTC compression
- Atlas budget: max 2048×2048 per category
- Export format: PNG with premultiplied alpha
```

### For 3D Artists — Model Spec

```markdown
## 3D Asset Spec: [Asset Name]

### Description
[What this model represents in-game]

### Technical Budget
| Parameter | Requirement |
|---|---|
| Triangle Count | [X – Y tris] |
| Texture Resolution | [256 / 512 / 1024] |
| UV Channels | [1 or 2] |
| LOD Levels | [0 / 1 / 2] |
| Rigged? | [Yes / No] |
| Animations | [List required clips] |

### Visual Reference
[Describe or link reference images]

### Optimization Notes
- Merge meshes where possible for draw call reduction
- Avoid alpha transparency on large surfaces
- Bake lighting into vertex colors if static
```

### For Developers — Technical Design Spec

```markdown
## Technical Spec: [Feature Name]

### Overview
[1 paragraph describing the feature from a technical perspective]

### Data Model
```csharp
// Pseudocode — adapt to project conventions
public interface IFeatureName
{
    void Execute(InputData data);
    event Action<ResultData> OnComplete;
}
```

### Algorithm
**Input**: [What data comes in]
**Process**: [Step-by-step logic]
**Output**: [What data comes out]

```
PSEUDOCODE:
1. Receive input X
2. Validate X against constraints
3. Calculate result = formula(X)
4. Emit OnComplete(result)
5. Update persistent state
```

### Performance Constraints
- Must run in < [X] ms per frame
- Zero GC allocation in hot path
- Object pool for any instantiated elements

### Edge Cases
| Condition | Expected Behavior |
|---|---|
| [Input is null] | [Return default, log warning] |
| [Value exceeds max] | [Clamp to max] |
```

## Best Practices
- Every spec must be **self-contained** — reader should not need to ask follow-up questions
- Include visual references (even rough sketches described in text) whenever possible
- Always cross-reference production specs from `gd-03-production-specs`
- For Dev specs: include both the happy path AND edge cases
- For Art specs: always include exact pixel dimensions and compression format
