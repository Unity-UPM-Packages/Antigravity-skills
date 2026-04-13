# Rule 05: Git & Code Versioning Conventions

## Overview
A strict standardization for managing version control, ensuring the Git history remains a clean, readable, and trackable narrative across the entire game project.

## 1. Commit Message Format (Conventional Commits)
- MUST strictly use the standard prefix format: `type(scope): subject`
- Types allowed:
  - `feat`: A new gameplay feature or system implementation.
  - `fix`: A bug fix.
  - `refactor`: A code change that neither fixes a bug nor adds a feature (e.g., solidifying logic according to `Rule 01`).
  - `ui`: Canvas/uGUI and graphical interface changes.
  - `perf`: A code improvement specifically focusing on Optimization (Zero GC fixes, GPU Draw Call reductions).
  - `chore`: Modifying `.gemini/` rules, Unity Project Settings, build process, or docs.
- **Example:** `feat(Inventory): implement modular chest system utilizing factory pattern`

## 2. Unity Versioning Safety
- `.meta` Awareness: Never commit an asset without its corresponding `.meta` file. Warning the user explicitly if an orphan meta file is detected.
- Never commit cache/engine-generated folders: `/Library/`, `/Temp/`, or `/Logs/`.
