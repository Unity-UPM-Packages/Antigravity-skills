---
name: 3d-rigging-animation
description: "[3D Artist] Use when rigging characters, setting up bone hierarchies, skinning weights, creating animation clips, or exporting animated models from Blender to Unity."
---

# Skill: Rigging & Animation

## When to use this skill
- User needs to rig a character for animation
- User asks about bone setup, weight painting, or IK
- User needs animation clips created or exported

## Step-by-Step Execution

### Step 1 — Rig Planning

```
CHARACTER: [Name]
RIG TYPE: [Humanoid / Generic / Simple (prop animation)]
BONE COUNT: [Target — mobile: 30–50 bones max]
IK NEEDED: [Yes / No — which limbs?]
UNITY RETARGET: [Humanoid (for Mecanim) / Generic]
```

### Step 2 — Bone Hierarchy (Humanoid Standard)

```
Root
└── Hips
    ├── Spine
    │   ├── Chest
    │   │   ├── Neck → Head
    │   │   ├── Shoulder.L → UpperArm.L → LowerArm.L → Hand.L
    │   │   │   └── [Finger bones if needed]
    │   │   └── Shoulder.R → UpperArm.R → LowerArm.R → Hand.R
    │   │       └── [Finger bones if needed]
    ├── UpperLeg.L → LowerLeg.L → Foot.L → Toe.L
    └── UpperLeg.R → LowerLeg.R → Foot.R → Toe.R
```

**Mobile optimization**: Skip finger bones unless close-up hand shots are needed (saves ~20 bones).

### Step 3 — Weight Painting Rules
- Every vertex must be influenced by **≤ 4 bones** (mobile GPU limit)
- Joint areas: smooth gradient between parent/child bones
- No stray weights (vertices influenced by unrelated bones)
- Test weights by posing extreme poses (T-pose → action pose)

### Step 4 — Animation Clip Table

| Clip Name | Duration | FPS | Loop | Root Motion | Notes |
|---|---|---|---|---|---|
| idle | 1.0–2.0s | 30 | Yes | No | Subtle breathing, weight shift |
| walk | 0.8s | 30 | Yes | Optional | 2 contact frames |
| run | 0.5s | 30 | Yes | Optional | More exaggerated than walk |
| attack_01 | 0.5s | 30 | No | No | Impact at frame 12 |
| hit_react | 0.3s | 30 | No | No | Additive blend possible |
| death | 1.5s | 30 | No | No | End pose = ragdoll start |
| victory | 2.0s | 30 | No | No | Celebration pose |

### Step 5 — Export Animated FBX

```
Armature:
  ✅ Only Deform Bones
  ❌ Add Leaf Bones (Unity doesn't need them)
Animation:
  ✅ Bake Animation (All Actions)
  ✅ NLA Strips (if using NLA workflow)
  Key Reduction: 0.1 (slight cleanup)
  Force Start/End Keying: ✅
```

### Step 6 — Unity Import Setup
```
Rig Tab:
  Animation Type: Humanoid (for retargetable) / Generic
  Avatar Definition: Create From This Model
  
Animation Tab:
  Per clip: Set Loop Time, Root Transform settings
  Events: Add AnimationEvents at impact frames
```

## Best Practices
- Always rig in **T-pose** or **A-pose** (A-pose deforms shoulders better)
- Name bones following Unity Humanoid convention for auto-mapping
- Bake IK to FK before export (Unity doesn't support Blender IK)
- Animation at 30fps is sufficient for mobile — 60fps doubles data for negligible visual gain
- Keep root bone at world origin (0,0,0) — avoids offset issues in Unity
