---
name: 2d-figma-designer-guide
description: "[2D Artist & UI/UX] Use when creating, reviewing, or auditing a Figma file intended for the Unity automation pipeline. Loads the project's design convention as the authoritative ruleset for naming, components, and structure."
---

# Skill: Figma Designer Guide

## When to Activate
- Creating a new UI screen in Figma for this project
- Reviewing a designer's Figma file before handoff to dev
- Using `2d-mcp-ui-composer` to build a Figma layout programmatically
- Answering questions about layer naming, component setup, constraints, or export rules

## Resource

**Load and treat as the primary reference before doing anything:**
`resources/Figma_Designer_Guide.md`

This document defines all naming conventions, component structure, variant rules, scroll view construction, constraints mapping, and the pre-handoff checklist. The automation pipeline breaks when these rules are violated — do not improvise.

## Execution

1. **Read** `resources/Figma_Designer_Guide.md` in full before starting
2. **Apply** every rule as-is — do not adapt or simplify
3. **Validate** against the checklist at the end of the guide before declaring the design ready
4. **Block handoff** if any checklist item fails — communicate which items need fixing

## Related Skills

| Situation | Skill |
|---|---|
| AI creates design in Figma via MCP | `2d-mcp-ui-composer` |
| UX review after creation | `2d-ux-audit` |
