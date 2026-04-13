---
name: git-commit
description: Use when the user asks to summarize diffs, write conventional git commit messages, or prepare work for pushing.
---

# Workflow: Automated Git Commit

## Objective
When the user asks the AI to "commit the current work" or "draft a commit message", utilize this workflow to eliminate the mental overhead of manually structuring git logs.

## Execution Sequence
1. **Context Initialization**: Review the modified files in the workspace (via the IDE diff tree).
2. **Rule Enforcement**: Map the analyzed changes strictly to `05-git-conventions.md` to pick the correct prefix (`feat`, `refactor`, `perf`, etc.).
3. **Drafting (Title & Body)**: 
   - Write a concise 50-character maximum Title.
   - If multiple files across different systems were altered, append a Body containing a bulleted list breaking down the "Why" and "What" of the architecture adjustments.
4. **Execution**: Provide the commit message clearly to the user, or if given explicit shell/system permission, automatically run the exact `git commit -m "..."` command to finalize the operation.

