# Skill: ALES Audit

## id
ales-audit

## applies_to
Checking structural consistency across multiple repos in an ALES ecosystem.

## Purpose
Compare the `agent-context/` structure across all repos in an ecosystem against the canonical ALES structure — surface gaps, inconsistencies, and drift so that all repos remain aligned and a future hub agent can navigate them predictably.

## When to Use
- Multiple repos in an ecosystem each have `agent-context/`.
- The request is any of: "audit ALES across repos", "check ALES consistency", "are all repos aligned", "ALES ecosystem check", "prepare repos for hub".
- Before connecting repos to a hub layer.
- After running `ales-update` on one repo, to verify others are still consistent.

---

## Steps

### Step 1 — IDENTIFY REPOS

List all repos in the ecosystem. For each repo, confirm:
- The repo root path
- Whether `agent-context/` exists

If a repo has no `agent-context/`, flag it as **NOT ALES-ENABLED** and recommend running `ales-bootstrap`.

### Step 2 — STRUCTURAL SCAN

For each ALES-enabled repo, check the following against the canonical structure in `reference/folder-structure.md`:

| Check | Expected | Finding |
|---|---|---|
| `intent/overview.md` | Present | PRESENT / MISSING |
| `intent/conventions.md` | Present | PRESENT / MISSING |
| `intent/anti-goals.md` | Present | PRESENT / MISSING |
| `intent/glossary.md` | Present | PRESENT / MISSING |
| `intent/decisions/` | Optional | PRESENT / ABSENT |
| `intent/dependencies/` | Present if cross-repo deps exist | PRESENT / ABSENT / NEEDED |
| `intent/dependencies/_index.md` | Present if folder exists | PRESENT / MISSING |
| `map/folders.md` | Present | PRESENT / MISSING |
| `map/workflow.md` | Present | PRESENT / MISSING |
| `tasks/refresh-map.task.md` | Present | PRESENT / MISSING |
| `templates/` folder | Present | PRESENT / MISSING |
| `plugins/` folder | Optional | PRESENT / ABSENT |
| `plugins/hub/manifest.md` | Present if Hub-connected | PRESENT / MISSING |

Also check for legacy patterns that should be migrated:
- `intent/vault.md` (flat file) → should be `intent/dependencies/vault.md`
- Any flat cross-repo alias file in `intent/` root → should be under `intent/dependencies/`

### Step 3 — CROSS-REPO CONSISTENCY CHECK

Compare shared conventions across repos:
- Do all repos reference the same Vault root path (if applicable)?
- Are branch naming conventions consistent across repos?
- Do repos that depend on each other have matching dependency contract files?

### Step 4 — HUB READINESS CHECK

Verify that a hub agent could navigate all repos without scanning:

| Hub operation | Requires | Status |
|---|---|---|
| Identify repo purpose | `intent/overview.md` | ✓/✗ per repo |
| Understand repo connections | `intent/dependencies/_index.md` | ✓/✗ per repo |
| Navigate repo structure | `map/folders.md` | ✓/✗ per repo |
| Know how to work in repo | `map/workflow.md` | ✓/✗ per repo |
| Expose Hub plugin manifest | `plugins/hub/manifest.md` | ✓/✗ per repo |

For each repo that has `plugins/hub/manifest.md`, run the Hub manifest validation checklist:
1. All required keys exist (`plugin_id`, `manifest_version`, `studio_id`, `studio_name`, `project_root_alias`, `status`, `current_work`, `blockers`, `recent_activity`, `bounds`, `security`).
2. All declared `source_path` values resolve under the repo root or a known alias.
3. `bounds` values are present and positive.
4. `security.allow_paths` and `security.deny_paths` are non-empty.
5. At least one of `status`, `current_work`, `blockers`, `recent_activity` source paths is readable.

Report each check as PASS / FAIL with a short reason.

### Step 5 — REPORT

```
ALES Ecosystem Audit — <date>

Repos scanned: <N>
ALES-enabled: <N>
Not ALES-enabled: <list>

Per-repo findings:
  <repo-name>:
    Status: ALIGNED | NEEDS UPDATE | LEGACY PATTERNS FOUND
    Missing: <list or "— none —">
    Legacy: <list or "— none —">
    Notes: <any cross-repo consistency issues>

Hub readiness:
  <table showing ✓/✗ per repo per hub operation>

Recommended actions:
  1. Run ales-update on: <list of repos with gaps>
  2. Migrate legacy patterns in: <list>
  3. Align dependency contracts between: <list of mismatched pairs>
```

---

## Constraints
- Read-only operation. This skill produces a report only — it does not create or modify files.
- To fix findings, run `ales-update` on the affected repo.

## Verification
- Every ALES-enabled repo has been checked against all items in Step 2.
- Hub readiness check covers all four required hub operations.

## Ask When
- A repo's root path is not known — ask before scanning.
- Two repos have conflicting Vault root paths — surface the conflict and ask which is correct before reporting.
