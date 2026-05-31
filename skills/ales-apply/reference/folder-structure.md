# ALES Canonical Folder Structure

> Version: 3.0 — Markdown-native maps, templates standard, cross-repo dependencies.

## Contents

- [Canonical Structure](#canonical-structure)
- [Layer Descriptions](#layer-descriptions)
- [File Formats](#file-formats)
  - [intent/overview.md](#intentoverviewmd)
  - [intent/conventions.md](#intentconventionsmd)
  - [intent/anti-goals.md](#intentanti-goalsmd)
  - [intent/glossary.md](#intentglossarymd)
  - [intent/decisions/](#intentdecisions)
  - [intent/dependencies/_index.md](#intentdependencies_indexmd)
  - [intent/dependencies/\<name\>.md](#intentdependenciesnamemd)
  - [map/folders.md](#mapfoldersmd)
  - [map/workflow.md](#mapworkflowmd)
  - [skills/\*.skill.md](#skillsskillmd)
  - [tasks/\*.task.md](#taskstaskmd)
  - [templates/\*.template.md](#templatestemplatemd)

---

## Canonical Structure

```
/agent-context/
├── intent/                         ← authored by humans; slow-changing
│   ├── overview.md                 ← what this repo is, its purpose, and core workflow
│   ├── conventions.md              ← naming, style, and process rules
│   ├── anti-goals.md               ← explicit non-goals; what this repo is NOT
│   ├── glossary.md                 ← domain terms and abbreviations
│   ├── decisions/                  ← ADR-style records for significant choices (optional)
│   │   └── <slug>.md
│   └── dependencies/               ← cross-repo and external system contracts (optional)
│       ├── _index.md               ← list of all dependencies; agents load this first
│       └── <dep-name>.md           ← one file per dependency (e.g. vault.md, hub.md)
│
├── map/                            ← maintained by agents; refreshed when source changes
│   ├── folders.md                  ← project folder layout with descriptions
│   ├── workflow.md                 ← core agent workflow and routing logic for this repo
│   └── <additional>.md             ← repo-specific map files (e.g. adapter-registry.md)
│
├── skills/                         ← project-specific how-to recipes
│   └── *.skill.md
│
├── tasks/                          ← portable agent task procedures
│   ├── refresh-map.task.md         ← always present; keeps map/ current
│   └── *.task.md
│
├── templates/                      ← starter files for humans and agents
│   └── *.template.md
│
└── plugins/                        ← optional; declared by third-party integrations
    └── <integration-id>/
        └── manifest.md             ← plugin manifest (contract between this repo and the integrator)
```

---

## Layer Descriptions

| Layer | Question it answers | Who owns it | Change frequency |
|---|---|---|---|
| `intent/` | *Why* is the system shaped this way? | Human (agent may draft) | Rarely — goals and architecture evolve slowly |
| `map/` | *Where* do things live and *what* exists right now? | Agent (derived from source) | TTL-based — refresh when source changes |
| `skills/` | *How do you do X specifically in this project?* | Human (agent may suggest) | Only when intent or conventions change |
| `tasks/` | *How does an agent execute a class of work?* | Spec / portable | Spec-versioned; reusable across repos |
| `templates/` | *What is the starter shape of this artifact?* | Human | When artifact format changes |
| `plugins/` | *What contract does this repo expose to integrators?* | Integration owner | When integration contract changes |

**Key distinction — skills vs tasks:**
A *task* is a generic, portable procedure (e.g., "refresh the map", "create a project").
A *skill* is a project-specific recipe that captures institutional knowledge (e.g., "how we structure specs in *this* repo").

---

## File Formats

### intent/overview.md

Free-form Markdown. Required sections:

| Section | Content |
|---|---|
| `## Purpose` | One paragraph: what this repo does and for whom |
| `## Core Workflow` | The canonical lifecycle, expressed as a short sequence |
| `## Principles` | 3–7 numbered rules that govern agent and human behavior |
| `## Agent Behavior` | Explicit rules for agents operating in this repo |

Optional sections: `## Relationship to …` for ecosystem context.

---

### intent/conventions.md

Markdown. Covers:
- Naming rules (files, folders, branches, commits)
- Code style references
- Process conventions (review gates, commit prefixes, etc.)

Keep every statement specific and falsifiable. No generic best practices.

---

### intent/anti-goals.md

Markdown. A list of explicit non-goals with short rationale for each.
Prevents scope creep and helps agents refuse off-scope requests confidently.

Format:
```markdown
## <Anti-goal label>
> Rationale: <one sentence>
```

---

### intent/glossary.md

Markdown. Term → definition pairs.
One term per heading. Each definition is one or two sentences, scoped to this repo.
Link to `decisions/` when a term has an associated ADR.

---

### intent/decisions/

One `.md` file per significant architectural or workflow decision.
Filename convention: `<YYYY-MM-DD>-<slug>.md` or `<NNN>-<slug>.md`.

Minimum required fields per file:
- `# Decision: <title>`
- `## Status` — `Proposed | Accepted | Superseded | Deprecated`
- `## Context` — why this decision was needed
- `## Decision` — what was decided
- `## Consequences` — what changes as a result

---

### intent/dependencies/_index.md

Markdown. Required by any repo that depends on another repo or external system.
Agents load this file before any cross-repo task.

Required table:

| Column | Content |
|---|---|
| `Dependency` | Name (links to the `.md` file) |
| `Type` | `repo` \| `service` \| `external` |
| `Direction` | `reads-from` \| `writes-to` \| `bidirectional` |
| `Purpose` | One sentence |

---

### intent/dependencies/\<name\>.md

One file per external dependency. Contains:
- `## Paths` — alias definitions and resolved paths
- `## Rules` — load-order rules, immutability constraints
- `## Branch Convention` — if the dependency uses git branches
- Any repo-specific cross-repo contract fields

---

### map/folders.md

Markdown. Describes every significant folder in the repo root.
Organized as: `## Root`, then named sections per top-level folder.
Each entry: `- \`path/\`: one-sentence description`.

Agents read this to navigate without scanning the filesystem.
Refresh when folders are added, renamed, or deleted.

---

### map/workflow.md

Markdown. Describes the canonical agent routing for this repo:
- The ordered lifecycle (e.g., `Brief → Corpus → Extract → Synthesize → Output`)
- Which task file handles each stage
- Decision points and branching rules

Agents read this to determine which task to load next.
Refresh when the lifecycle or task set changes.

---

### skills/\*.skill.md

One file per recurring project-specific recipe.
**Every statement must be specific to this repo** — no generic best practices.

Required sections:

```markdown
# Skill: <Human-readable name>

## id
<kebab-case-id>

## applies_to
<task or workflow this skill supports>

## Purpose
One sentence: what project-specific knowledge this skill captures.

## When to Use
- Specific condition that makes this skill relevant

## Steps
1. Concrete step — reference real paths, commands, or patterns

## Constraints
- Project-specific rules that must not be violated

## Verification
- How to confirm the step was done correctly in this project

## Ask When
- Condition where the agent must stop and ask rather than guess
```

---

### tasks/\*.task.md

Portable procedure files. Installed at the spec level; consistent across repos.

Required sections:

```markdown
# Task: <Human-readable name>

## id
<kebab-case-id>

## Goal
One sentence: the class of work this task handles.

## Load
Files to load before starting, in priority order:
1. agent-context/intent/overview.md
2. <additional files specific to this task>

## Steps
1. Step description

## Expected Output
What a successful execution produces.

## Stop Conditions
- When the agent must stop and report rather than guess
```

---

### templates/\*.template.md

Starter file skeletons. Filled in by humans or agents when creating a new artifact.
Filename: `<artifact-type>.template.md`.
Contents: the artifact structure with `<placeholder>` fields and inline guidance comments.

---

### plugins/\<integration-id\>/manifest.md

Optional. Present only when this repo participates in a third-party integration (e.g. a Hub agent).
One folder per integration. The `manifest.md` is the sole contract file.

Required fields for Hub integrations (`plugin_id: hub`):
- `plugin_id`, `manifest_version`, `studio_id`, `studio_name`, `project_root_alias`
- `status`, `current_work`, `blockers`, `recent_activity` — each with `source_path` and `read_mode`
- `bounds` — `max_files_per_query`, `max_lines_per_file`, `max_depth`
- `security` — `allow_paths` and `deny_paths` lists

**Rules:**
- Agents must never create or overwrite `plugins/*/manifest.md` without explicit integration-owner approval.
- Agents running `ales-update` must preserve the entire `plugins/` tree unchanged.
- `ales-audit` must detect and report plugin manifests as part of hub-readiness checks.
