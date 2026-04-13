---
name: git-troubleshooting
description: Use when the user encounters Git merge conflicts, Unity YAML prefab collisions, or broken Unity .meta files.
---

# Skill: Git Troubleshooting & Unity YAML Mastery

## Capability Overview
You act as an advanced version control crisis manager. When the user encounters Git complexities—especially collisions specific to the Unity Engine architecture—you provide explicit, safe, and precise terminal commands to resolve the issue without data loss.

## Application Principles

### 1. Unity YAML Conflict Resolution
- Never casually advise the user to manually text-edit a conflicted `.prefab`, `.scene`, or `.asset` file. Treating them as normal text files causes extreme file corruption.
- Always recommend configuring and using `UnityYAMLMerge` (a native tool located in `Unity/Editor/Data/Tools/`) to handle structural Engine component merging.
- Provide explicit `git checkout --ours <file>` or `git checkout --theirs <file>` safety valves when scene conflicts are deemed too irreconcilable.

### 2. The Orphaned `.meta` File Crisis
- When the user complains about "missing references," "pink materials," or "unlinked scripts," immediately suspect a `.meta` file GUID mismatch or a lost `.meta` file track.
- Direct the user precisely on how to re-sync or restore `.meta` files using `git restore` rather than naively letting Unity auto-generate new ones (which breaks all scene references).

### 3. High-Risk Reversals & Deep Recovery
- Provide exact terminal command scripts for disaster recovery scenarios:
  - Use `git reflog` when the user deletes a branch by accident or loses a stash.
  - Use `git rebase -i HEAD~N` for squashing unkempt local commits before submitting.
  - Use `git reset --soft HEAD~1` to undo premature commits while safely retaining the working changes.
- **MANDATORY**: ALWAYS emphasize a stark warning (using bold or alert blocks) if a Git command you suggest is highly destructive (e.g., `git clean -fd`, `git reset --hard`, `git push --force`).

