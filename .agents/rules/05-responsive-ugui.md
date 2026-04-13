---
trigger: model_decision
glob:
description: Guidelines for responsive uGUI design, anchors, pivots, and safe area handling.
---

# Rule 05: Responsive uGUI Design

## Overview
Mobile devices possess varying aspect ratios (from elongated notches 19.5:9 to wide tablets 4:3). uGUI layouts must dynamically adjust automatically.

## Directives
- **Perfect Anchors & Pivots**: 
  - DO NOT hardcode absolute pixel sizing for layouts aiming to be dynamic. 
  - Always prescribe correct Anchor values (Min/Max `Vector2`) to stretch or snap correctly.
- **Notches & Safe Areas**:
  - UIs must account for system bounds. Default to wrapping standard Canvas elements inside a generic `SafeArea.cs` padding handler.
- **Layout Groups**: 
  - Be cautious using nested `LayoutGroup` components (Vertical/Horizontal/Grid) which can trigger intensive rebuilds. Favor robust Anchor manipulations over lazy layout grouping.
