---
name: git-workflow
description: Use when committing changes, writing commit messages, resolving Git conflicts, recovering from Git mistakes, auditing Unity YAML or .meta file issues, or managing branches in a Unity project.
---

# Skill: Git Workflow — Commit, Branch & Recovery

## Capability Overview
The agent autonomously manages the full Git lifecycle for a Unity project: from scanning what changed and generating conventional commit messages, to resolving Unity-specific YAML conflicts and recovering from destructive mistakes. The agent acts as both an automated commit writer and a version-control crisis manager.

---

## Phase 1 — Context Scan (Always Run First)

Before any Git action, scan the workspace state:

```
1. What files changed? (modified, added, deleted, renamed)
2. Which systems/features do the changes belong to?
3. Are there any Unity-specific file types involved? (.scene, .prefab, .asset, .meta)
4. Is the working tree clean or are there uncommitted Unity scene changes?
5. Is the current branch appropriate for this type of change?
```

**Output of this scan** → assign each changed file to a `scope` and `type` for the commit message.

---

## Phase 2 — Commit Workflow

### 2.1 Conventional Commit Format
Cross-reference `rules/06-git-conventions.md` to assign the correct prefix:

| Prefix | When to Use | Example |
|---|---|---|
| `feat` | New gameplay feature or system | `feat(inventory): add item stacking logic` |
| `fix` | Bug fix, crash, incorrect behavior | `fix(health): clamp damage to prevent negative HP` |
| `refactor` | Code restructure, no behavior change | `refactor(player): extract ShootSystem from PlayerController` |
| `perf` | Performance improvement | `perf(ui): isolate HUD into separate Canvas` |
| `test` | Add or fix unit tests | `test(inventory): add edge case for full capacity` |
| `chore` | Build config, package updates, tooling | `chore: update Unity version to 6000.0.23` |
| `docs` | Documentation, comments | `docs(readme): update architecture overview` |
| `style` | Formatting, naming conventions only | `style: apply Allman brace convention to WeaponSystem` |
| `art` | Asset changes (sprites, audio, prefabs) | `art(enemies): add slime idle animation frames` |

### 2.2 Commit Message Construction

**Title rules** (agent enforces these automatically):
- Maximum **50 characters**
- Format: `type(scope): short imperative description`
- No period at end
- Scope = the system/feature affected (`player`, `inventory`, `ui`, `audio`)

**Body rules** (add when > 2 files changed across different systems):
- Explain **Why**, not What (the diff already shows What)
- Bullet list per logical change
- Wrap at 72 characters

```
feat(weapons): add modular firing system

- Extract IWeaponSystem interface to decouple Player from weapon logic
- Implement FirearmWeapon and MeleeWeapon as separate strategies
- Wire via GameBootstrapper — PlayerController now injection-ready
- Resolves tight coupling detected in code review (Issue #42)
```

### 2.3 Auto-Commit Execution Sequence
When user says "commit" or "push this":

1. **Scan** modified files (`git status`, `git diff --name-only`)
2. **Group** by system/scope
3. **Draft** commit message following 2.2 format
4. **Present** message to user for confirmation (unless explicitly told to auto-run)
5. **Execute**: `git add .` → `git commit -m "title" -m "body"` (if approved)
6. **Report**: confirm commit hash and files committed

### 2.4 Staging Strategy
Never blindly `git add .` for large multi-system changes. Split into atomic commits:

```bash
# Wrong: one massive commit for everything
git add . && git commit -m "feat: lots of stuff"

# Correct: atomic commits per logical unit
git add Scripts/Systems/Inventory/ && git commit -m "feat(inventory): add stack merging"
git add Scripts/Views/Inventory/ && git commit -m "feat(inventory-ui): bind stack count to cell view"
```

Agent decision: if the diff spans > 3 unrelated systems → propose splitting into multiple commits.

---

## Phase 3 — Unity-Specific Conflict Resolution

### 3.1 YAML Scene/Prefab Conflicts

**Critical Rule**: NEVER manually text-edit a conflicted `.scene`, `.prefab`, or `.asset` file as plain text. The YAML structure is fragile — manual edits cause silent corruption that breaks references without obvious errors.

**Correct approach — UnityYAMLMerge**:
```bash
# Configure once in .gitconfig
[mergetool "unityyamlmerge"]
    cmd = 'C:/Program Files/Unity/Hub/Editor/<VERSION>/Editor/Data/Tools/UnityYAMLMerge.exe' merge -p "$BASE" "$REMOTE" "$LOCAL" "$MERGED"
    trustExitCode = false

[merge]
    tool = unityyamlmerge
```

Then resolve conflicts:
```bash
git mergetool  # Invokes UnityYAMLMerge automatically for .scene / .prefab files
```

**If conflict is irreconcilable** (e.g., two developers built entirely different versions of the same scene):
```bash
# Keep your version entirely
git checkout --ours Assets/Scenes/GameplayScene.unity
git add Assets/Scenes/GameplayScene.unity

# OR keep the incoming version entirely
git checkout --theirs Assets/Scenes/GameplayScene.unity
git add Assets/Scenes/GameplayScene.unity
```

Then communicate to the team which changes were dropped and need re-applying.

### 3.2 Orphaned .meta File Crisis
**Symptoms**: "Missing Script" errors, pink/magenta materials, broken Prefab references, serialized field nulls.

**Root cause diagnosis**:
```bash
# Find .meta files with no corresponding asset (orphaned)
git status  # Look for deleted assets that still have .meta files tracked

# Find assets that have no .meta file (new assets not committed)
git ls-files --others --exclude-standard | grep -v .meta
```

**Fix protocol**:
```bash
# If .meta was accidentally deleted — restore it (never let Unity auto-regenerate)
git restore Assets/Scripts/Systems/InventorySystem.cs.meta

# If a new asset was added without committing the .meta — stage both together
git add Assets/Art/Icons/sword_icon.png Assets/Art/Icons/sword_icon.png.meta
```

> ⚠️ **Never** let Unity auto-regenerate a `.meta` for a file that had an existing `.meta` in git. Auto-generation creates a new GUID — breaking every scene/prefab that referenced the old GUID.

### 3.3 .gitattributes for Unity (Mandatory Configuration)
Ensure this is in the project root `.gitattributes`:

```
# Unity YAML files — force text mode for UnityYAMLMerge
*.unity    merge=unityyamlmerge eol=lf
*.prefab   merge=unityyamlmerge eol=lf
*.asset    merge=unityyamlmerge eol=lf
*.meta     merge=unityyamlmerge eol=lf

# Binary assets — force binary mode (no diff/merge)
*.png      binary
*.jpg      binary
*.fbx      binary
*.mp3      binary
*.wav      binary
*.ogg      binary
```

If this file is missing from the project — generate it immediately.

---

## Phase 4 — Disaster Recovery

### 4.1 Lost Commit / Deleted Branch
```bash
# Find lost commits via reflog
git reflog | head -20  # Shows all recent HEAD movements

# Recover the commit
git checkout <commit-hash>          # Inspect it
git branch recovery/lost-feature    # Save it as a new branch
```

### 4.2 Undo Last Commit (Keep Changes)
```bash
# Undo the commit, keep all changes staged
git reset --soft HEAD~1

# Undo the commit, keep changes in working tree (unstaged)
git reset --mixed HEAD~1  # Default behavior
```

### 4.3 Squash Messy Local Commits Before Pushing
```bash
# Squash last 4 commits into one clean commit
git rebase -i HEAD~4
# In the editor: keep first as 'pick', mark rest as 'squash'
```

### 4.4 Stash Rescue
```bash
git stash list                  # List all stashes
git stash show -p stash@{0}     # Preview contents
git stash pop                   # Apply + remove latest stash
git stash apply stash@{2}       # Apply specific stash without removing
```

### 4.5 Destructive Command Safety Protocol

> **⚠️ WARNING — The following commands can cause PERMANENT data loss.** The agent MUST display this warning block and NEVER auto-execute these without explicit user confirmation.

| Command | Risk | Safe Alternative |
|---|---|---|
| `git reset --hard` | Destroys all uncommitted work | `git stash` first |
| `git clean -fd` | Permanently deletes untracked files | `git clean -n` to preview first |
| `git push --force` | Rewrites remote history, breaks teammates | `git push --force-with-lease` instead |
| `git rebase` on shared branch | Rewrites history others depend on | Only rebase local/feature branches |

**Agent rule**: If asked to run any of the above, ALWAYS:
1. Show the warning
2. Explain the risk for this specific situation
3. Propose the safer alternative
4. Execute ONLY after explicit "yes, proceed" from user

---

## Phase 5 — Proactive Git Health Checks

The agent proactively flags these situations without being asked:

| Signal | Agent Action |
|---|---|
| Uncommitted Unity scene files before a major task | Warn: "You have unsaved scene changes — commit or stash before proceeding?" |
| `.meta` file missing for a new asset | Flag immediately: "Asset added without .meta — must be committed together" |
| Commit message in non-conventional format | Rewrite and propose corrected message |
| Branch has > 20 commits ahead of main | Suggest: "Consider squashing before merge request" |
| `.gitattributes` missing from project | Generate and commit it |

---

## Cross-Skill References
- Summarizing architecture changes for commit body → `skills/code-review`
- Identifying what system/scope changed → `skills/design-pattern-architect`
