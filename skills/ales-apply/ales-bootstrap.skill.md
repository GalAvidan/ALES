# Skill: ALES Bootstrap

## id
ales-bootstrap

## applies_to
Setting up ALES on a project that has no `agent-context/` folder yet.

## Purpose
Scaffold a complete, populated `/agent-context/` folder from scratch so that agents can immediately work with bounded, deterministic context.

## When to Use
- The target repo has no `agent-context/` folder.
- The request is any of: "apply ALES", "set up ALES", "bootstrap agent context", "add /agent-context", "initialize ALES".

---

## Steps

### Step 1 — DISCOVER

Scan the project root. Collect:

| What to look for | Used for |
|---|---|
| `README.md`, `README.*` | Project description and purpose |
| `package.json`, `pyproject.toml`, `*.csproj`, `go.mod`, `Cargo.toml` | Primary language and stack |
| `src/`, `lib/`, `app/`, `packages/`, `projects/` | Module layout |
| `.eslintrc*`, `.prettierrc*`, `ruff.toml`, `tsconfig.json` | Conventions source |
| `AGENTS.md`, `BOT.md`, `CLAUDE.md`, `CONTEXT.md` | Existing agent adapter files |
| Other repos referenced or aliased in config files | Dependencies |

Record inferred values: `project_name`, `primary_language`, `framework`, `module_roots[]`, `has_vault_dependency`, `has_other_dependencies`.

### Step 2 — CONFIRM

Present inferred values to the user:

```
Inferred:
  project_name:      <value>
  primary_language:  <value>
  module_roots:      <list>
  dependencies:      <list or "none detected">

Please confirm or correct. Also provide:
  - One-sentence description of what this project does
```

Wait for confirmation before writing anything.

### Step 3 — SCAFFOLD

Create the folder structure:

```
agent-context/
├── intent/
│   └── dependencies/        ← only if dependencies detected
├── map/
├── skills/
├── tasks/
└── templates/
```

### Step 4 — POPULATE INTENT

Create `intent/` files using templates from `reference/file-templates.md`:

- `intent/overview.md` — derive purpose and workflow from README and source structure; confirm core workflow with user if unclear
- `intent/conventions.md` — derive from linting configs, existing naming patterns, and any AGENTS.md or CONTEXT.md files
- `intent/anti-goals.md` — derive from README scope sections or ask user for 2–3 non-goals
- `intent/glossary.md` — derive domain terms from README, task files, and source identifiers

If dependencies were detected:
- `intent/dependencies/_index.md` — list each detected dependency
- `intent/dependencies/<dep-name>.md` — one file per dependency, using path aliases from config files

### Step 5 — POPULATE MAP

Create map files using templates from `reference/file-templates.md`:

- `map/folders.md` — document every significant top-level folder with one-sentence descriptions
- `map/workflow.md` — express the core agent lifecycle and which task handles each stage

### Step 6 — INSTALL STARTER TASKS

Create `tasks/refresh-map.task.md` using the template in `reference/file-templates.md`.
Add any task files that already exist in the repo (e.g., project-specific task `.md` files) to the task folder if they are not already there.

### Step 7 — GENERATE SKILLS

Inspect project source and generate `skills/*.skill.md` files for project-specific recipes.

Good skill candidates:

| Skill slug | Derive from |
|---|---|
| `naming-conventions.skill.md` | Existing file/folder/symbol names |
| `module-structure.skill.md` | How modules, packages, or services are laid out |
| `add-feature.skill.md` | Patterns for adding new endpoints, components, or modules |
| `testing-conventions.skill.md` | Test file locations, naming, runner commands, mock patterns |
| `build-and-run.skill.md` | Build commands, dev server commands, environment setup |

Use the skill file format in `reference/folder-structure.md`.

### Step 8 — REPORT

```
ALES bootstrap complete.

Created:
  agent-context/intent/overview.md
  agent-context/intent/conventions.md
  agent-context/intent/anti-goals.md
  agent-context/intent/glossary.md
  [agent-context/intent/dependencies/_index.md]
  [agent-context/intent/dependencies/<dep-name>.md]
  agent-context/map/folders.md
  agent-context/map/workflow.md
  agent-context/tasks/refresh-map.task.md
  agent-context/skills/<list of generated skills>

Next steps:
  1. Review intent/ — correct anything the agent got wrong.
  2. Review skills/ — edit any that misrepresent actual project conventions.
  3. Run refresh-map after significant source changes.
```

---

## Constraints
- Never write any file before Step 2 confirmation.
- Never overwrite existing files. If `agent-context/` partially exists, use `ales-update` instead.
- Every generated file must reference real paths, real commands, or real patterns found in this repo. No generic content.

## Verification
- `agent-context/intent/overview.md` exists and has a non-placeholder `## Purpose` section.
- `agent-context/map/folders.md` lists every top-level folder.
- `agent-context/tasks/refresh-map.task.md` is present.

## Ask When
- The project purpose cannot be inferred from README or source — ask for a one-sentence description.
- Two or more files imply conflicting conventions — show the conflict and ask which is authoritative.
- A dependency is detected but its path or role is unclear — ask before creating the dependency file.
