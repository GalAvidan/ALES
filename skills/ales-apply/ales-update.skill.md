# Skill: ALES Update

## id
ales-update

## applies_to
Updating or expanding an existing `agent-context/` folder to match the current canonical structure.

## Purpose
Bring an existing `/agent-context/` installation current with the latest ALES structure — adding missing folders and files, refreshing stale map content, and extending intent files with new information — without ever overwriting human-authored content.

## When to Use
- `agent-context/` already exists in the target repo.
- The request is any of: "update ALES", "refresh agent context", "sync ALES", "add missing ALES files", "ALES is out of date".
- After a significant structural change to the project (new top-level folders, new lifecycle stages, new dependencies).

---

## Steps

### Step 1 — LOAD CURRENT STATE

Read the existing `agent-context/` structure. Note:
- Which folders exist vs. which are missing from the canonical structure
- Which files exist in each folder
- The content of `intent/overview.md` (to understand the project)

### Step 2 — DIFF

Compare the existing structure against the canonical structure in `reference/folder-structure.md`.

Build a diff covering:

| Check | Finding |
|---|---|
| Missing folders | Required folders not present |
| Missing files | Required files absent from present folders |
| Missing dependencies folder | Any cross-repo aliases in intent files not yet in `intent/dependencies/` |
| Missing `templates/` folder | Present in canonical but absent here |
| Stale map content | Map files whose source folders have been added, renamed, or deleted |

### Step 3 — CONFIRM

Show the diff to the user before writing anything:

```
Update diff for agent-context/:

  Missing folders:
    agent-context/intent/dependencies/   (cross-repo aliases detected in intent/)
    agent-context/templates/

  Missing files:
    agent-context/tasks/refresh-map.task.md

  Map files to refresh:
    agent-context/map/folders.md         (new top-level folders detected)

No existing files will be modified. Proceed? [y/N]
```

Wait for explicit confirmation.

### Step 4 — ADD MISSING

Create any missing folders and files using templates from `reference/file-templates.md`.
**Never overwrite existing files.** This step is strictly additive.

If `intent/vault.md` or any flat cross-repo alias file is found directly in `intent/`:
- Create `intent/dependencies/` folder.
- Move the content into `intent/dependencies/<dep-name>.md`.
- Create `intent/dependencies/_index.md` referencing it.
- Note this migration in the report.

### Step 5 — REFRESH MAP

For each map file flagged as stale:
1. Re-read the current folder structure.
2. Update `map/folders.md` entries: add new folders, mark removed folders as deprecated.
3. Update `map/workflow.md` if the lifecycle or task set has changed.
4. **Never remove human-authored context** — only add new entries or update factual paths.

### Step 6 — REPORT

```
ALES update complete.

Added:
  <list of new files and folders created>

Migrated:
  intent/vault.md → intent/dependencies/vault.md   (if applicable)
  <any other migrations>

Refreshed:
  <list of map files updated>

Unchanged:
  <list of files that were already current>

Review recommended:
  <Any files the agent updated that contain human-authored content>
```

---

## Constraints
- Never overwrite or truncate any existing file.
- Every addition must use templates from `reference/file-templates.md` as a base.
- Migration of `intent/vault.md` (or similar flat cross-repo files) to `intent/dependencies/` is additive — create the new file, then remove the old one only after confirming with the user.

## Verification
- All folders in the canonical structure exist under `agent-context/`.
- `agent-context/tasks/refresh-map.task.md` is present.
- If cross-repo aliases existed in `intent/`, they now live under `intent/dependencies/`.

## Ask When
- A map file is stale but the current folder structure is ambiguous — ask before updating.
- A dependency file migration would result in a naming conflict — ask which name to use.
- An existing intent file has content that seems contradicted by current source — surface the conflict, don't resolve it unilaterally.
