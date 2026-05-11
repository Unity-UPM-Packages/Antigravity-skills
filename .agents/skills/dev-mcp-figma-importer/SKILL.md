---
name: dev-mcp-figma-importer
description: "[Developer] Use when importing a Figma design into Unity as uGUI prefabs and scene objects. Covers the full automation pipeline: validation → prefab build → screen build → smart merge → font pipeline."
---

# Skill: Figma → Unity Automation Pipeline

## When to Activate
- User provides a Figma file URL, node ID, or Figma API data for Unity import
- Re-importing a Figma screen that already exists in Unity (smart merge required)
- Answering questions about how specific Figma elements map to Unity components

## Resource

**Load as the primary technical specification before executing anything:**
`resources/Figma_To_Unity_Pipeline.md`

All mapping rules, component structures, and property values are defined there. Do not guess or improvise — deviations produce incorrect output.

## Execution Order

Execute strictly in this sequence. All detail lives in the resource document.

| Step | Action | Resource Section |
|---|---|---|
| 1 | **Validate** — run the pre-import checklist; halt on failure | Section 17 |
| 2 | **Build Prefabs** — process all Master Components first | Section 3 |
| 3 | **Build Screens** — process each `Screen_*` frame | Section 16 |
| 4 | **Smart Merge** — if screen exists, preserve custom MonoBehaviours | Section 4 |
| 5 | **Font Pipeline** — resolve Google Fonts → create TMP assets | Section 13 |
| 6 | **Report** — output import summary with creates / updates / warnings | Section 4.4 |

## Non-Negotiable Rules

These are easy to miss and cause silent, hard-to-detect bugs:

| Rule | Reference | Why it matters |
|---|---|---|
| Reverse sibling order when creating GameObjects | Section 16.2 | Figma and Unity have opposite z-ordering |
| Never delete a GO that has custom scripts | Section 4.3 | Destroys serialized field bindings permanently |
| `sprite.border = Vector4(L, B, R, T)` — **not** `(L, R, T, B)` | Section 7.3 | Unity border order differs from annotation order |
| Build all Master Component Prefabs before any Screen | Section 3 | Nested Prefab references break if Screens are built first |

## Related Skills

| Situation | Skill |
|---|---|
| Build Unity objects from prompt or image (not Figma) | `dev-mcp-unity-builder` |
| Need an Editor Script fallback | `dev-editor-scripting` |
| Post-import Canvas performance review | `dev-ui-performance` |
