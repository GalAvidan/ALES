---
name: ales-apply
description: 'Applies ALES (Agent-Layer Execution Specification) to any project by scaffolding or updating the /agent-context/ folder — the structured knowledge contract that lets LLM agents work with bounded context and auditable traversal. Use when setting up ALES on a new project, applying ALES to an existing codebase, bootstrapping agent context, adding /agent-context, initializing ALES, refreshing agent context, or updating an existing ALES structure.'
---

# Applying ALES

Scaffolds or updates the `/agent-context/` folder in any project using the ALES v2 specification.
Gives any LLM agent the structured knowledge it needs to work in this project without loading the entire codebase.

Two modes, auto-detected:

| Mode | Trigger |
|---|---|
| **Bootstrap** | `/agent-context/` does not exist → create from scratch |
| **Update** | `/agent-context/` already exists → diff, extend, and refresh |

Reference files:
- [reference/folder-structure.md](reference/folder-structure.md) — canonical structure, file formats
- [reference/file-templates.md](reference/file-templates.md) — ready-to-fill templates for every generated file

---

## When to Use

- "Apply ALES to this project"
- "Bootstrap agent context" / "Set up ALES" / "Initialize ALES"
- "Add /agent-context to this project"
- "Update ALES" / "Refresh agent context" / "Update my agent context"
- Starting a project and wanting agents to work effectively without flooding context

---

## Detect Mode

1. Check if `/agent-context/ales.manifest.json` exists.
2. **Present** → **UPDATE** mode.
3. **Absent** → **BOOTSTRAP** mode.

Tell the user which mode was detected before proceeding.

---

## Bootstrap Procedure

### Step 1 — DISCOVER

Scan the project root. Collect:

| What to look for | Used for |
|---|---|
| `README.md` or `README.*` | Project description source |
| `package.json`, `pyproject.toml`, `*.csproj`, `go.mod`, `Cargo.toml` | Primary language and stack |
| `src/`, `lib/`, `app/`, `packages/`, `projects/` | Module layout |
| `.eslintrc*`, `.prettierrc*`, `ruff.toml`, `tsconfig.json` | Conventions source |
| `openapi.yaml`, `swagger.json`, `*.proto` | API surface source |

Record inferred values: `project_name`, `primary_language`, `framework`, `module_roots[]`.

### Step 2 — CONFIRM

Present inferred values to the user:

```
Inferred:
  project_name:      <value>
  primary_language:  <value>
  module_roots:      <list>

Please confirm or correct. Also provide:
  - One-sentence description of what this project does
```

Wait for confirmation or corrections before writing anything.

### Step 3 — SCAFFOLD

Create folders:

```
/agent-context/
├── map/
├── skills/
└── tasks/
```

### Step 4 — POPULATE MAP

Derive map files using schemas in `reference/folder-structure.md` and templates in `reference/file-templates.md`.

- `map/modules.json` — one entry per detected module root; set `path`, `type`, `description` from folder name and contents
- `map/apis.json` — derive entries from any found API surface files; write `[]` if none found
- `map/_provenance.json` — write fresh provenance: `generated_at` = now, `stale: false`, hash each `derived_from` source

### Step 5 — CREATE MANIFEST

Write `ales.manifest.json` using the template in `reference/file-templates.md`. Set:

- `ales_version`: `"2.0"`
- `project_name`: confirmed value
- `primary_language`: confirmed value
- `created_at`: current ISO 8601 timestamp
- `updated_at`: same as `created_at`
- `repo_sha`: output of `git rev-parse HEAD`; use `"UNKNOWN"` if not a git repo

### Step 6 — INSTALL STARTER TASKS

Write the two starter task files using templates from `reference/file-templates.md`:

- `tasks/refresh-map.task.md`
- `tasks/check-staleness.task.md`

### Step 7 — GENERATE SKILLS

Inspect project source and generate `skills/*.skill.md` files that capture project-specific recipes.
Skills are AI-authored — derive them from real patterns found in this codebase, not generic best practices.

Good skill candidates (generate whichever apply):

| Skill slug | Derive from |
|---|---|
| `naming-conventions.skill.md` | Existing file/folder/symbol names across the codebase |
| `module-structure.skill.md` | How modules, packages, or services are laid out and wired together |
| `add-feature.skill.md` | Patterns for adding new endpoints, components, or modules |
| `testing-conventions.skill.md` | Test file locations, naming, runner commands, mock patterns |
| `build-and-run.skill.md` | Build commands, dev server commands, environment setup |

Use the skill file format in `reference/folder-structure.md`.

### Step 8 — REPORT

```
ALES bootstrap complete.

Created:
  ales.manifest.json
  map/modules.json
  map/apis.json
  map/_provenance.json
  skills/<list of generated skills>
  tasks/refresh-map.task.md
  tasks/check-staleness.task.md

Next steps:
  1. Review skills/ — edit any that misrepresent actual project conventions.
  2. Run check-staleness after significant source changes.
  3. Run refresh-map when module layout changes.
```

---

## Update Procedure

### Step 1 — DISCOVER

Read `ales.manifest.json`. Note:

- `ales_version` — check against current spec (2.0)
- `updated_at` — compute and show age
- `repo_sha` — compare against `git rev-parse HEAD`

### Step 2 — DIFF

Compare existing `/agent-context/` against the canonical structure in `reference/folder-structure.md`.

Build a diff report covering:

- **Missing folders** — required folders that do not exist
- **Missing files** — required files absent from existing folders
- **Stale map entries** — entries in `map/_provenance.json` where `stale: true` or TTL has elapsed
- **Outdated manifest fields** — fields present in v2.0 spec but missing from the manifest

### Step 3 — CONFIRM

Show the diff report before writing anything:

```
Update diff for /agent-context/:

  Missing files:
    tasks/check-staleness.task.md

  Stale map entries:
    map/modules.json  (generated: <date>, <N> days ago)

  Manifest updates:
    updated_at → <current timestamp>
    repo_sha   → <current sha>

No existing files will be overwritten. Proceed? [y/N]
```

Wait for explicit confirmation.

### Step 4 — ADD MISSING

Create any missing folders and files from the canonical structure.
**Never overwrite existing files.** This step is additive only.

### Step 5 — REFRESH MAP

For each map file whose provenance is stale or whose TTL has elapsed:

1. Re-derive content from current source (same logic as Bootstrap Step 4).
2. Update `map/_provenance.json`: set `generated_at` = now, recompute `fingerprint`, set `stale: false`, clear `stale_reason`.

### Step 6 — UPDATE SKILLS

Compare existing `skills/` against current source patterns.
For any recurring pattern not yet captured in an existing skill file:

1. Generate a new `*.skill.md` file for it.
2. **Never overwrite existing skill files** — additive only.

### Step 7 — UPDATE MANIFEST

Write updated fields to `ales.manifest.json`:

- `updated_at` → current ISO 8601 timestamp
- `repo_sha` → current git SHA
- `ales_version` → `"2.0"` (or latest if spec changed)

### Step 8 — REPORT

```
ALES update complete.

Added:
  tasks/check-staleness.task.md
  skills/<any new skills>

Refreshed:
  map/modules.json
  map/_provenance.json

Unchanged:
  <list unchanged files>

Still stale (requires manual source inspection):
  <list or "— none —">
```

---

## Verification

After Bootstrap or Update, confirm:

1. `ales.manifest.json` is valid JSON with all required fields.
2. Every `map/*.json` entry has a `_meta` block with `stale: false`.
3. Every `skills/*.skill.md` has an `id`, an `applies_to` line, and at least one numbered step.
4. Every `tasks/*.task.md` has a title and a Steps section.
5. During Update: no existing files were overwritten (check file timestamps).
