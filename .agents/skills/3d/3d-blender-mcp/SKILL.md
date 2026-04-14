---
name: 3d-blender-mcp
description: "[3D Artist] Use when connecting to Blender via MCP to create, modify, or export 3D models programmatically. Can be combined with Unity MCP for cross-tool workflows."
---

# Skill: Blender MCP Integration

## When to use this skill
- User asks to create or modify a model directly in Blender
- User wants to automate repetitive Blender operations
- A Blender MCP server is available and connected

## Overview
This skill is **tool-agnostic in implementation** — it describes the operations the AI should perform through whatever Blender MCP is available. Specific tool names depend on the connected MCP server.

## Pre-flight Check
1. **Verify Blender MCP connection**: Check if a Blender MCP server is active
2. **If NOT connected**: Fall back to generating Blender operation instructions in text
3. **If connected**: Proceed with workflow below

## Step-by-Step Execution

### Step 1 — Scene Setup
```
Operations:
1. Clear default scene (remove default cube, camera, light)
2. Set unit system: Metric, Scale = 1.0
3. Set viewport: Perspective, 3D cursor at origin
```

### Step 2 — Model Operations
```
Common operations:
- Create primitive mesh (cube, sphere, cylinder, plane)
- Add/apply modifiers (Subdivision, Mirror, Decimate, Solidify)
- Edit mesh (extrude, loop cut, merge vertices)
- UV unwrap (Smart UV Project or manual seam-based)
- Assign materials
- Set object origin
- Rename objects following naming convention
```

### Step 3 — Rig Operations (if needed)
```
Operations:
- Create armature
- Add bones following hierarchy from 3d-rigging-animation skill
- Parent mesh to armature (Automatic Weights)
- Adjust weight painting
```

### Step 4 — Export
```
Operations:
1. Select exportable objects
2. Apply all transforms
3. Export FBX with settings from 3d-modeling skill
4. Output file path for Unity import step
```

## Cross-MCP Workflow: Unity → Blender
When user wants to edit an existing Unity asset in Blender:
```
1. [Unity MCP] Locate the FBX asset in the Unity project
   → Get file path: Assets/Art/Characters/chr_player.fbx
2. [Blender MCP] Open the FBX file in Blender
3. [Blender MCP] Perform requested modifications
4. [Blender MCP] Export overwrite to same file path
5. [Unity MCP] Refresh AssetDatabase to reimport
```

## Error Handling
| Situation | Fallback |
|---|---|
| Blender MCP not connected | Output Blender instructions as text (step-by-step guide) |
| Model too complex for automated editing | Describe changes needed, let user do manually |
| FBX export fails | Check scale, transforms, modifiers — report diagnostics |

## Best Practices
- Always work on a copy if modifying production assets
- Verify export settings match `3d-modeling` skill checklist before every export
- After cross-MCP edit, always verify in Unity that import settings are preserved
