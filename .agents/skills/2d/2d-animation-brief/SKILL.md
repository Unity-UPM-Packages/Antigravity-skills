---
name: 2d-animation-brief
description: "[2D Artist & UI/UX] Use when specifying animations — both frame-by-frame sprite sheet animations and Spine skeletal animations. Also covers UI micro-interactions and transition motion."
---

# Skill: Animation Brief

## When to use this skill
- User needs animation specs for characters, enemies, or VFX
- User asks about UI transitions or micro-interactions
- User needs to choose between frame-by-frame vs Spine

## Decision Tree

| Content Type | Recommended Method | Rationale |
|---|---|---|
| Simple UI feedback (button press, pop) | **CSS-style tween** (scale, alpha, position) | No asset needed, code-driven |
| UI transitions (screen enter/exit) | **Tween sequence** (code or DOTween) | Lightweight, easily adjustable |
| Character with few actions (idle, walk) | **Frame-by-frame sprite sheet** | Simple pipeline, no rigging needed |
| Character with many actions + blend | **Spine / DragonBones** | Skeletal = less memory, smooth blending |
| VFX / Particle explosions | **Frame-by-frame + Particle System** | Best visual impact |
| Cutscene / Dialogue expressions | **Spine** | Facial rig flexibility |

## Step-by-Step Execution

### For Frame-by-Frame Animations

```markdown
## Animation: [Character/Object Name]

### Sprite Sheet Spec
| Field | Value |
|---|---|
| Sheet Size | [POT — e.g., 1024×1024] |
| Frame Size | [e.g., 128×128 per frame] |
| Total Frames | [N] |

### Clip Table
| Clip | Frames | FPS | Loop | Notes |
|---|---|---|---|---|
| idle | 1–6 | 8 | Yes | Subtle breathing motion |
| run | 7–14 | 12 | Yes | Contact frames at 7, 11 |
| attack | 15–22 | 16 | No | Impact frame at 19 — trigger hit event |
| hurt | 23–26 | 12 | No | Flash white on frame 23 |
| death | 27–34 | 10 | No | Fade out alpha on final 3 frames |

### Timing Notes
- idle: 750ms total cycle
- attack: anticipation (3f) → swing (2f) → impact (1f) → recovery (2f)
```

### For Spine / Skeletal Animations

```markdown
## Spine Rig: [Character Name]

### Bone Structure
- Root
  ├── Hip
  │   ├── Torso → Chest → Head
  │   ├── UpperArm_L → LowerArm_L → Hand_L
  │   ├── UpperArm_R → LowerArm_R → Hand_R
  │   ├── UpperLeg_L → LowerLeg_L → Foot_L
  │   └── UpperLeg_R → LowerLeg_R → Foot_R
  └── Weapon_Slot (attachable)

### Animation List
| Animation | Duration | Loop | Blend Priority |
|---|---|---|---|
| idle | 1.0s | Yes | Base layer |
| walk | 0.6s | Yes | Base layer |
| attack_01 | 0.5s | No | Override layer |
| hit_react | 0.3s | No | Additive layer |
| death | 1.2s | No | Override (highest) |

### Mesh Deformation
- Face: mouth, eyes (2 deform keys each)
- Chest: breathing deformation on idle
```

### For UI Micro-Interactions

```markdown
## UI Motion: [Screen/Element Name]

| Interaction | Property | From → To | Duration | Easing | Delay |
|---|---|---|---|---|---|
| Button press | Scale | 1.0 → 0.95 | 80ms | EaseOut | 0ms |
| Button release | Scale | 0.95 → 1.0 | 120ms | EaseOutBack | 0ms |
| Screen enter | Position Y | +100% → 0% | 300ms | EaseOutCubic | 0ms |
| Screen exit | Alpha | 1.0 → 0.0 | 200ms | Linear | 0ms |
| Reward pop | Scale | 0.0 → 1.2 → 1.0 | 400ms | EaseOutElastic | 0ms |
| Notification badge | Scale | 0.0 → 1.0 | 250ms | EaseOutBack | 100ms |
| Counter increment | Text value | Old → New | 500ms | Linear | 0ms |
```

## Best Practices
- **12 Principles of Animation** apply to games too — especially Anticipation, Follow-through, and Squash & Stretch
- Always define the **impact frame** (the exact frame where gameplay events trigger: damage dealt, projectile spawned)
- Spine rigs should limit to **30 bones** for mobile performance
- Frame-by-frame at 24fps is luxury — 12fps with good keyframes looks great and saves 50% memory
