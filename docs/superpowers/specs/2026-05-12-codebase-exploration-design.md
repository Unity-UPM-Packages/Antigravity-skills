# Codebase Exploration Skill - Design Spec

## 1. Overview
This design outlines a new AI skill: `dev-codebase-exploration`.
Its primary goal is to provide a highly structured, token-efficient methodology for the AI to explore, read, and understand a Unity project. It prevents the AI from aimlessly reading large files or irrelevant third-party packages, which causes massive token burn and context dilution.

## 2. Trigger Conditions
This skill activates automatically when the user requests the AI to:
- "Understand this project"
- "Explain how module X works"
- "Review the overall codebase"
- Or when the AI is dropped into a new project without prior context.

*(Note: It does not trigger for feature creation or bug fixing where the target files are already known).*

## 3. Core Directives (Non-Negotiable Constraints)
- **Token Protection:** By default, the AI MUST NOT read or search within `Packages/`, `Library/`, or `Plugins/` directories unless explicitly instructed or investigating a related compile error.
- **Breadth-First Exploration:** The AI must understand the high-level architecture (directory structures, file names) before diving into the detailed logic of any `.cs` file.
- **Contract-Driven Analysis:** The AI must prioritize reading Interfaces (`I*.cs`) and Data Models (`ScriptableObject`, `struct`, `enum`) to understand module contracts before reading the concrete implementation logic.
- **YAML Handling (Prefabs, Scenes, Assets):** The AI MUST NOT use `view_file` to read the entirety of `.prefab`, `.unity`, or `.asset` files. It must strictly use `grep_search` to look for specific component names, variables, or `m_FileID` references.
- **ProjectSettings Scope:** The AI is permitted to read files inside `ProjectSettings/`, but ONLY when the task directly relates to rendering, physics, input, or build pipeline issues.

## 4. Execution Workflow (5-Step Optimal Path)
When activated, the AI must strictly follow this execution order:
1.  **High-Level Scan:** Use `list_dir` on `Assets/` (and main code folders) to identify if the project uses a feature-based or layer-based architecture.
2.  **Architecture Check:** Read `README.md` and `Assembly Definition` (`.asmdef`) files to map out module dependencies. Do NOT read code yet.
3.  **Entry Point Discovery:** Search for execution roots (e.g., `Bootstrapper`, `GameManager`, `MainInstaller`, `AppFlow`). Read these to understand the initialization flow.
4.  **Interface & Contract Review:** Locate the target feature area. Open and read ONLY the Interfaces and Data Structures to understand the inputs/outputs.
5.  **Targeted Deep Dive:** Only after the contract is understood, open the concrete implementation files (`.cs`) specifically relevant to the user's inquiry.

## 5. Output Format (Reporting Standards)
After the exploration, the AI must respond to the user with:
1.  **Architecture Diagram:** A `Mermaid` diagram illustrating the relationship between the discovered modules/classes.
2.  **Flow Summary:** A concise explanation of the execution flow and data lifecycle.
3.  **Blind Spots Declaration:** An explicit statement acknowledging what was deliberately skipped (e.g., "I skipped reading the UI prefabs and the Networking package to save tokens. Let me know if you want to dive into them.").

## 6. Next Steps
Once this specification is approved by the user, the actual skill file (`.agents/skills/dev-codebase-exploration/SKILL.md`) will be generated.
