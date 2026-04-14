---
name: 2d-design-to-unity
description: "[2D Artist & UI/UX] Use when exporting finished assets from a design tool and importing them into Unity via MCP. Handles the full pipeline: export settings → file organization → Unity import → scene placement."
---

# Skill: Design-to-Unity Pipeline

## When to use this skill
- User has approved a design in the design tool and says "export this"
- User wants to push assets directly into Unity
- User asks to update existing Unity UI with new design assets

## Pre-flight Check
1. **Design Tool MCP**: Connected? (for export)
2. **Unity MCP**: Connected? (for import)
3. If either is missing → fall back to manual export instructions

## Step-by-Step Execution

### Step 1 — Export from Design Tool
```
Operations:
1. Select all exportable frames/components
2. Apply export settings per asset type:
   - UI Sprites: PNG-24, @2x resolution, trim transparent pixels
   - Backgrounds: JPG quality 90% (or PNG if alpha needed)
   - 9-Slice elements: PNG-24, document slice borders
   - Icons: PNG-24, @2x, individual files
3. Export to a staging folder organized by category:
   /export/
   ├── ui_common/
   ├── ui_shop/
   ├── characters/
   ├── backgrounds/
   └── icons/
```

### Step 2 — Pre-import Validation
Before pushing to Unity, verify:

| Check | Status |
|---|---|
| All files named per `2d-03-production-standards` convention | ☐ |
| Texture dimensions are POT (or acceptable NPOT for atlas-packed) | ☐ |
| 9-slice border values documented | ☐ |
| No duplicate filenames across folders | ☐ |
| Total size within budget (check atlas estimates) | ☐ |

### Step 3 — Import into Unity (via Unity MCP)
```
Operations:
1. Create/verify Unity folder structure:
   Assets/
   ├── Art/
   │   ├── UI/
   │   │   ├── Common/
   │   │   ├── Shop/
   │   │   └── HUD/
   │   ├── Characters/
   │   └── Backgrounds/
2. Copy exported files to corresponding Unity folders
3. Set TextureImporter settings per asset type:
   - Texture Type: Sprite (2D and UI)
   - Sprite Mode: Single (or Multiple for sheets)
   - Packing Tag: [atlas name]
   - Max Size: [from asset spec]
   - Compression: ASTC
   - Filter: Bilinear
4. For 9-slice sprites: Set sprite borders in TextureImporter
5. Refresh AssetDatabase
```

### Step 4 — Scene Placement (Optional)
If user requests direct UI building:
```
Operations:
1. Create/find Canvas in target scene
2. Instantiate UI elements from imported sprites
3. Apply RectTransform positioning per wireframe spec
4. Set anchors for responsive behavior
5. Attach placeholder scripts (if specified)
```

### Step 5 — Report
Output a summary:
```markdown
## Import Report

### Assets Imported
| Asset | Unity Path | Import Settings | Status |
|---|---|---|---|
| btn_primary_default | Assets/Art/UI/Common/ | Sprite, ASTC 4×4, 512 | ✅ |
| ... | ... | ... | ... |

### Scene Updates
- [List any GameObjects created or modified]

### Warnings
- [Any issues encountered during import]
```

## Fallback (No MCP Available)
If Unity MCP is not connected, output:
1. Folder structure instructions (where to place files manually)
2. A TextureImporter preset `.preset` file if possible
3. Step-by-step manual import guide

## Best Practices
- Always import into a version-controlled project (commit before bulk import)
- Set Packing Tags BEFORE importing to avoid double-process
- Never overwrite existing assets without confirming with user first
- After import, run a quick build to verify no broken references
