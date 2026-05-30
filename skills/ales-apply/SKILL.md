---
name: ales-apply
description: >
  Applies ALES (Agent-Layer Execution Specification) to any project by scaffolding or
  updating the /agent-context/ folder — the structured knowledge contract that lets LLM
  agents work with bounded context and auditable traversal.
  Routes to the correct sub-skill based on context.
  Use when: setting up ALES on a new project, applying ALES to an existing codebase,
  bootstrapping agent context, adding /agent-context, initializing ALES, refreshing agent
  context, updating an existing ALES structure, or auditing consistency across an ecosystem.
---

# ALES Apply — Entry Point

This skill routes to one of three sub-skills depending on context.

## Sub-Skills

| Sub-skill | File | When to use |
|---|---|---|
| **ales-bootstrap** | [ales-bootstrap.skill.md](ales-bootstrap.skill.md) | No `agent-context/` exists in this repo |
| **ales-update** | [ales-update.skill.md](ales-update.skill.md) | `agent-context/` already exists; needs refresh or expansion |
| **ales-audit** | [ales-audit.skill.md](ales-audit.skill.md) | Check structural consistency across multiple repos in an ecosystem |

## Reference Files

- [reference/folder-structure.md](reference/folder-structure.md) — canonical structure, file formats, and file descriptions
- [reference/file-templates.md](reference/file-templates.md) — ready-to-fill templates for every generated file

---

## Routing Decision

1. Check if `agent-context/` exists in the target repo root.
2. **Absent** → load and follow `ales-bootstrap.skill.md`.
3. **Present** → load and follow `ales-update.skill.md`.
4. If the request is about comparing multiple repos or checking ecosystem consistency → load and follow `ales-audit.skill.md`.

Tell the user which sub-skill was selected before proceeding.
