---
trigger: always_on
glob:
description: Conventional Commits format, atomic commits, .meta file safety, and .gitattributes requirements for Unity projects.
---

# Rule 06: Git & Code Versioning Conventions

## Overview
A strict standardization for managing version control, ensuring the Git history remains a clean, readable, and trackable narrative across the entire game project. All commit messages must follow the Conventional Commits standard.

## 1. Commit Message Format

**Format**: `type(scope): short imperative description`
- Title: maximum **50 characters**, no period at end
- Scope: the system or feature affected — e.g., `player`, `inventory`, `ui`, `audio`, `build`
- Body (optional): explain **Why**, not What. Add when > 2 unrelated files changed.

## 2. Allowed Commit Types

| Type | When to Use | Example |
|---|---|---|
| `feat` | New gameplay feature or system | `feat(inventory): add item stacking logic` |
| `fix` | Bug fix, crash, incorrect behaviour | `fix(health): clamp damage to prevent negative HP` |
| `refactor` | Code restructure, no behaviour change | `refactor(player): extract ShootSystem from PlayerController` |
| `perf` | Performance improvement | `perf(ui): isolate HUD into separate Canvas` |
| `test` | Add or fix unit/integration tests | `test(inventory): add edge case for full capacity` |
| `chore` | Build config, package updates, tooling, `.agents/` rule changes | `chore: update Unity to 6000.0.23f1` |
| `docs` | Documentation, comments, README | `docs(readme): update architecture overview` |
| `style` | Formatting, naming conventions only — no logic change | `style: apply Allman brace style to WeaponSystem` |
| `art` | Asset changes: sprites, audio, prefabs, animations | `art(enemies): add slime idle animation frames` |

> **Removed**: `ui` type — use `feat`, `fix`, or `style` for UI changes with the `ui` scope instead. Example: `feat(ui): add shop screen canvas structure`

## 3. Unity Versioning Safety

- **`.meta` Awareness**: Never commit an asset without its corresponding `.meta` file. Flag orphaned `.meta` files immediately — auto-regeneration creates a new GUID and breaks all scene/prefab references.
- **Never commit** cache or engine-generated folders: `/Library/`, `/Temp/`, `/Logs/`, `/Builds/`
- **`.gitattributes` required**: Unity YAML files (`.unity`, `.prefab`, `.asset`) must use `merge=unityyamlmerge eol=lf`. Binary assets (`.png`, `.fbx`, `.mp3`) must be marked `binary`. See `skills/git-workflow` for the full `.gitattributes` template.
- **Atomic commits**: If changes span > 3 unrelated systems, split into separate commits — one per logical change.
