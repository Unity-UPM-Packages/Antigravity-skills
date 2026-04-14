---
name: 2d-design-tool-mcp
description: "[2D Artist & UI/UX] Use when connecting to an external design tool (Figma, Photoshop, or any compatible tool) via MCP to create frames, components, and layouts programmatically."
---

# Skill: Design Tool MCP Integration

## When to use this skill
- User asks to create a UI directly in a design tool
- User wants to automate frame creation, component placement, or style application
- A Design Tool MCP server is available and connected

## Overview
This skill is **tool-agnostic** — it describes the operations the AI should perform through whatever Design Tool MCP is available. The specific MCP tool names will vary depending on the connected server.

## Pre-flight Check
Before attempting any MCP operation:
1. **Verify MCP connection**: Check if a Design Tool MCP server is active
2. **If NOT connected**: Fall back to generating a text-based wireframe spec (use `2d-ui-wireframe` skill instead)
3. **If connected**: Proceed with the workflow below

## Step-by-Step Execution

### Step 1 — Project Setup
```
Operations to perform:
1. Create a new page/artboard named "[ProjectName]-UI"
2. Set canvas size to target resolution (1080×1920 for mobile portrait)
3. Create a shared styles/tokens library:
   - Color styles from art direction palette
   - Text styles from typography scale
   - Spacing constants from grid system
```

### Step 2 — Frame Creation
For each screen in the wireframe spec:
```
Operations:
1. Create a frame "[ScreenName]" at target resolution
2. Add SafeArea padding (top: 44dp, bottom: 34dp)
3. Create layout structure:
   - Header region (top)
   - Content region (center, scrollable if needed)
   - Footer / Bottom nav (bottom, fixed)
4. Place elements according to wireframe layout
5. Apply styles from the shared library
```

### Step 3 — Component Building
For reusable elements:
```
Operations:
1. Create main component "[ComponentName]"
2. Define variants:
   - State: Default, Pressed, Disabled, Loading
   - Size: Small, Medium, Large (if applicable)
3. Apply auto-layout / responsive constraints
4. Document usage notes in component description
```

### Step 4 — Review Checkpoint
After creating the design:
```
1. List all created frames and components
2. Present a summary to the user for review
3. Wait for user approval before proceeding to export
```

## Error Handling
| Situation | Fallback |
|---|---|
| MCP not connected | Generate text wireframe spec instead |
| MCP tool call fails | Retry once → if fail again, report error and fallback to text |
| Design tool version incompatible | Report capability gap, suggest manual steps |

## Best Practices
- Always create components, never one-off shapes (enables reuse)
- Apply the project's design tokens instead of hardcoded values
- Name every layer and frame following `2d-03-production-standards` conventions
- After creation, always run a quick UX audit (`2d-ux-audit`) on the result
