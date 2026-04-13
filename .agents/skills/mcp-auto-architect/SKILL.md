# Skill: Unity MCP Auto-Architect

## Capability Overview
Grants the AI the technical understanding needed to orchestrate commands to a third-party Unity MCP Server dynamically. Acts as the execution bridge between reasoning loops and engine interactions.

## Application Principles
- **API Navigation**: Comprehend exactly which node paths, instantiation APIs, or property editors are exposed by the MCP endpoint.
- **Component Association**: When creating game structures (like an Inventory Panel), ensure `Image`, `Button`, or `TextMeshProUGUI` component properties are correctly pushed through identical mapping contexts.
- **Pipeline Trust**: Ensure tasks executed through the MCP successfully serialize to `.prefab` or `.scene` architectures and notify the developer upon successful binding tasks.
