---
name: dev-codebase-exploration
description: Use when the user asks to "understand the project", "explain how a module works", "review the overall codebase", or when exploring a new project without prior context. This skill enforces strict, token-efficient procedures for reading Unity projects by defaulting to Assets, ignoring third-party packages, and using targeted searches for heavy YAML files.
---

# Skill: Codebase Exploration

This skill ensures you explore Unity projects efficiently without burning excessive tokens on irrelevant third-party code or massive YAML files.

## 1. Core Directives (Non-Negotiable)

- **Token Protection:** By default, you MUST NOT use search or read tools within `Packages/`, `Library/`, or `Plugins/` directories unless explicitly instructed or investigating a related compile error.
- **Breadth-First Exploration:** Understand the high-level architecture (directory structures, file names) before reading the detailed logic of any `.cs` file.
- **Contract-Driven Analysis:** Prioritize reading Interfaces (`I*.cs`) and Data Models (`ScriptableObject`, `struct`, `enum`) to understand module contracts *before* reading concrete implementation logic.
- **YAML Handling (Prefabs, Scenes, Assets):** You MUST NOT use `view_file` to read the entirety of `.prefab`, `.unity`, or `.asset` files. You must strictly use `grep_search` to look for specific component names, variables, or `m_FileID` references.
- **ProjectSettings Scope:** You are permitted to read files inside `ProjectSettings/`, but ONLY when the task directly relates to rendering, physics, input, or build pipeline issues.

## 2. Execution Workflow (The 5-Step Optimal Path)

When asked to explore or understand a codebase, follow these steps strictly in order:

### Step 1: High-Level Scan
Use `list_dir` on `Assets/` (and main source folders like `Assets/Scripts/`) to identify if the project uses a feature-based or layer-based architecture. Do not read any files yet.

### Step 2: Architecture Check
Read `README.md` and `Assembly Definition` (`.asmdef`) files to map out module dependencies. Do not read C# code yet.

### Step 3: Entry Point Discovery
Search for execution roots (e.g., `Bootstrapper.cs`, `GameManager.cs`, `MainInstaller.cs`, `AppFlow.cs`). Read these files to understand the initialization flow.

### Step 4: Interface & Contract Review
Locate the target feature area the user is asking about. Open and read ONLY the Interfaces and Data Structures to understand the inputs and outputs. Do not read the implementation classes.

### Step 5: Targeted Deep Dive
Only after the contract is understood, open the concrete implementation files (`.cs`) specifically relevant to the user's inquiry.

## 3. Output Format (Reporting Standards)

After exploration, you MUST respond to the user with the following structure:

1. **Architecture Diagram:** Create a `Mermaid` diagram illustrating the relationship between the discovered modules/classes.
2. **Flow Summary:** Provide a concise explanation of the execution flow and data lifecycle.
3. **Blind Spots Declaration:** Explicitly state what you deliberately skipped (e.g., "I skipped reading the UI prefabs and the Networking package to save tokens. Let me know if you want to dive into them.").
