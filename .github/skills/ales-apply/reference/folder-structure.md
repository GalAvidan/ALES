## Contents

- [Canonical Structure](#canonical-structure)
- [File Descriptions](#file-descriptions)
- [Skill File Format](#skill-file-format)
- [Task File Format](#task-file-format)

---

## Canonical Structure

```
/agent-context/
├── ales.manifest.json          ← spec version, project metadata, repo sha, timestamps
├── map/
│   ├── modules.json            ← derived map of project modules and folder layout
│   ├── apis.json               ← derived map of exposed APIs and endpoints
│   └── _provenance.json        ← fingerprints and TTL metadata for each map file
├── skills/
│   └── *.skill.md              ← AI-generated project-specific recipes and conventions
└── tasks/
    ├── refresh-map.task.md     ← procedure for re-deriving and updating map/ files
    └── check-staleness.task.md ← procedure for detecting and reporting stale map entries
```

---

## File Descriptions

### `ales.manifest.json`

Root contract. Loaded first by every agent. Required fields:

| Field | Type | Description |
|---|---|---|
| `ales_version` | string | Spec version this manifest targets (`"2.0"`) |
| `schema_version` | string | Schema version; matches `ales_version` |
| `project_name` | string | Human-readable project name |
| `primary_language` | string | Primary programming language (`"TypeScript"`, `"Python"`, `"C#"`, etc.) |
| `created_at` | ISO 8601 | When ALES was first applied to this project |
| `updated_at` | ISO 8601 | Last time the manifest or map/ was refreshed |
| `repo_sha` | string | Git commit SHA at last update; `"UNKNOWN"` if not a git repo |
| `source_of_truth_order` | string[] | Precedence list for conflicting facts |
| `task_registry` | object[] | List of installed tasks; each entry has `id` and `file` |

---

### `map/modules.json`

Agent-maintained. Derived from source. An array of module entries:

| Field | Type | Description |
|---|---|---|
| `id` | string | Kebab-case identifier for the module |
| `type` | string | `"package"`, `"service"`, `"library"`, `"app"`, `"test"`, `"config"` |
| `path` | string | Relative path from project root |
| `description` | string | One sentence: what this module does |
| `_meta` | object | Provenance block (see `_provenance.json`) |

---

### `map/apis.json`

Agent-maintained. Derived from source. An array of API surface entries. Write `[]` if the project exposes no APIs.

| Field | Type | Description |
|---|---|---|
| `id` | string | Kebab-case identifier |
| `type` | string | `"rest"`, `"graphql"`, `"grpc"`, `"event"`, `"cli"` |
| `path` | string | Source file or spec file where this API is defined |
| `summary` | string | One sentence: what this API does |
| `_meta` | object | Provenance block |

---

### `map/_provenance.json`

Master provenance index. Tracks staleness for every derived map file.

| Field | Type | Description |
|---|---|---|
| `generated_at` | ISO 8601 | When this provenance block was last written |
| `ttl_days` | number | Days before this entry should be refreshed (default: 7) |
| `derived_from` | string[] | Glob patterns of source files this entry was derived from |
| `fingerprint` | string | Hash of the combined source content at generation time |
| `stale` | boolean | `true` if source has changed since last generation |
| `stale_reason` | string\|null | Human-readable reason when stale; `null` otherwise |

---

### `skills/*.skill.md`

One file per recurring workflow type. AI-generated from project source analysis.
**Every statement must be specific to this project** — no generic best-practice content.

---

### `tasks/*.task.md`

Portable procedure files. Two starters are always installed:

- `refresh-map.task.md` — re-derive and update `map/` files
- `check-staleness.task.md` — detect and report stale map entries

---

## Skill File Format

```markdown
# Skill: <Human-readable name>

## id
<kebab-case-id>

## applies_to
<task or workflow this skill supports, e.g. "add-feature", "debugging", "naming">

## Purpose
One sentence describing exactly what project-specific knowledge this skill captures.

## When to Use
- Specific condition that makes this skill relevant
- Another specific condition

## Steps
1. <Concrete, project-grounded step — reference real paths, commands, or patterns>
2. <Next step>

## Constraints
- Project-specific rules that must not be violated
- Reference real files or conventions found in this repo

## Verification
- How to confirm the step was completed correctly in this project

## Ask When
- Condition where the agent must stop and ask a human rather than guess
```

---

## Task File Format

```markdown
# Task: <Human-readable name>

## id
<kebab-case-id>

## Goal
One sentence describing the class of work this task handles.

## Context Priority
Files to load, in order:
1. ales.manifest.json
2. map/modules.json
3. <additional files specific to this task>

## Steps
1. <Step description>
2. <Next step>

## Expected Output
Description of what a successful execution produces.

## Stop Conditions
- Condition under which the agent must stop and report rather than guess
```
