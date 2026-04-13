---
name: wireframe-to-unity-mcp
description: Use when the user provides a wireframe or skeleton image and asks to auto-generate the graphical layout visually in the Unity Editor.
---

# Workflow: Wireframe to Unity uGUI via MCP

## Objective
Convert a provided visual wireframe/mockup directly into instantiated native Unity GameObjects in the scene/prefab utilizing the configured Unity MCP Server.

## Execution Sequence
1. **Visual Scan**: Analyze the wireframe layout using vision capabilities. Break down the components top-down (Canvas -> Containers -> Headers -> Grids -> Elements).
2. **Responsive Constraint Matrix**: Map `04-responsive-ugui.md` constraints to the identified Containers. Determine appropriate `Anchors`, `Pivots`, and `Offsets` to ensure responsiveness across devices.
3. **Performance Allocation**: Enforce `ui-performance.md` logic (e.g., isolating static background images vs highly dynamic scrolling lists) into distinct Canvas branches.
4. **MCP Command Execution**: Invoke the Unity MCP Server tool calls to execute construction inside the Unity editor natively. (i.e. Automatically creating `GameObjects`, assigning `RectTransforms`, attaching `Image`/`Text` components).
5. **System Output Checkout**: Validate the constructed system structure and ensure it integrates cleanly as an independent prefab logic module.

