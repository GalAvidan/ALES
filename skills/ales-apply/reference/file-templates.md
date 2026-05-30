# ALES File Templates

Ready-to-fill starters for every file generated during Bootstrap or Update.
Replace all `<placeholder>` values before writing.

## Contents

- [intent/overview.md](#intentoverviewmd)
- [intent/conventions.md](#intentconventionsmd)
- [intent/anti-goals.md](#intentanti-goalsmd)
- [intent/glossary.md](#intentglossarymd)
- [intent/dependencies/_index.md](#intentdependencies_indexmd)
- [intent/dependencies/\<name\>.md](#intentdependenciesnamemd)
- [map/folders.md](#mapfoldersmd)
- [map/workflow.md](#mapworkflowmd)
- [tasks/refresh-map.task.md](#tasksrefresh-maptaskmd)

---

## intent/overview.md

```markdown
# Overview

## Purpose

<One paragraph: what this repo does, for whom, and what value it delivers.>

## Core Workflow

```
<Step 1>  →  <Step 2>  →  <Step 3>  →  <Step N>
```

## Principles

1. <Guiding rule — specific to this repo>
2. <Guiding rule>
3. <Guiding rule>

## Agent Behavior

- Load only the context the current task names in its `Load:` block.
- Ask one focused question when intent is ambiguous. Do not guess.
- <Any repo-specific agent rule>
```

---

## intent/conventions.md

```markdown
# Conventions

## Naming

- Files: <rule>
- Folders: <rule>
- Branches: `<prefix>/<slug>` — e.g. `<example>`
- Commits: `<prefix>(scope): description` — e.g. `<example>`

## Code Style

- <Language/tool>: <rule or config file reference>

## Process

- <Review gate rule>
- <When to create an ADR vs just implement>
```

---

## intent/anti-goals.md

```markdown
# Anti-Goals

## <Anti-goal label>
> Rationale: <One sentence explaining why this is out of scope.>

## <Anti-goal label>
> Rationale: <One sentence.>
```

---

## intent/glossary.md

```markdown
# Glossary

## <Term>
<One or two sentence definition scoped to this repo.>

## <Term>
<Definition.>
```

---

## intent/dependencies/_index.md

```markdown
# Dependencies

This repo depends on the following external repos and systems.
Load this file before any cross-repo task, then load the specific dependency file for the repo you need.

| Dependency | Type | Direction | Purpose |
|---|---|---|---|
| [<dep-name>](<dep-name>.md) | `<repo\|service\|external>` | `<reads-from\|writes-to\|bidirectional>` | <One sentence> |
```

---

## intent/dependencies/\<name\>.md

```markdown
# <Dependency Name>

<One sentence: what this dependency is.>

## Paths

<alias>: <resolved path or URL>
<alias>: <resolved path>

## Rules

- <Load-order rule: "Load this file before any task that reads or writes <alias> paths.">
- <Immutability rule if applicable>
- <Any other contract rule>

## Branch Convention

- <Branch prefix and pattern for work that affects this dependency>

## Notes

- <Any other repo-specific cross-repo contract detail>
```

---

## map/folders.md

```markdown
# Folder Map

## Root

- `<file>`: <one-sentence description>
- `<file>`: <one-sentence description>

## <Top-level folder>

- `<subfolder>/`: <one-sentence description>
- `<subfolder>/`: <one-sentence description>

## <Top-level folder>

- `<subfolder>/`: <one-sentence description>
```

---

## map/workflow.md

```markdown
# Workflow

## Core Lifecycle

```
<Stage 1>  →  <Stage 2>  →  <Stage 3>  →  <Stage N>
```

The order is fixed. <Any lifecycle constraint>.

## Stage Routing

| Stage | Task file | Entry condition |
|---|---|---|
| <Stage 1> | `tasks/<task>.task.md` | <When to enter this stage> |
| <Stage 2> | `tasks/<task>.task.md` | <When to enter this stage> |

## Decision Points

- **<Branch name>**: <Condition> → <Which path to take>
```

---

## tasks/refresh-map.task.md

```markdown
# Task: Refresh Map

## id
refresh-map

## Goal
Re-derive all `map/` files from current project source and keep them current.

## Load
1. `agent-context/intent/overview.md`
2. `agent-context/map/folders.md`

## Steps
1. Scan the project root for added, renamed, or deleted top-level folders and files.
2. Update `map/folders.md` entries to reflect current state. Add new entries; mark removed entries as deprecated or delete them.
3. Check `map/workflow.md` — if the lifecycle or task set has changed, update accordingly.
4. Check any repo-specific map files and update if their source has changed.
5. Report which files were changed and which were unchanged.

## Expected Output
All `map/*.md` files accurately describe the current project structure and workflow.

## Stop Conditions
- A folder's purpose is ambiguous — stop and ask the human before writing.
- A task file referenced in `workflow.md` no longer exists — stop and report.
```
