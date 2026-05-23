## Contents

- [ales.manifest.json](#alesmanifesjson)
- [map/modules.json](#mapmodeulesjson)
- [map/apis.json](#mapapisjson)
- [map/_provenance.json](#map_provenancejson)
- [tasks/refresh-map.task.md](#tasksrefresh-maptaskmd)
- [tasks/check-staleness.task.md](#taskscheck-stalenessstaskmd)

---

## ales.manifest.json

Replace all `<placeholders>` before writing.

```json
{
  "ales_version": "2.0",
  "schema_version": "2.0",
  "project_name": "<project_name>",
  "primary_language": "<TypeScript|Python|C#|Go|Rust|other>",
  "description": "<one sentence: what this project does>",
  "created_at": "<ISO 8601 timestamp>",
  "updated_at": "<ISO 8601 timestamp>",
  "repo_sha": "<git rev-parse HEAD or UNKNOWN>",
  "source_of_truth_order": [
    "tests and runtime behavior",
    "production code",
    "ADRs and architecture docs",
    "README and developer docs",
    "agent-authored map entries"
  ],
  "task_registry": [
    { "id": "refresh-map",        "file": "tasks/refresh-map.task.md" },
    { "id": "check-staleness",    "file": "tasks/check-staleness.task.md" }
  ]
}
```

---

## map/modules.json

One entry per detected module root. Add or remove entries to match the project.

```json
[
  {
    "id": "<kebab-case-module-id>",
    "type": "<package|service|library|app|test|config>",
    "path": "<relative/path/from/project/root>",
    "description": "<one sentence: what this module does>",
    "_meta": {
      "generated_at": "<ISO 8601 timestamp>",
      "ttl_days": 7,
      "derived_from": ["<glob pattern of source files, e.g. src/module-name/**>"],
      "fingerprint": "<sha256:hash-of-source-content-at-generation>",
      "stale": false,
      "stale_reason": null
    }
  }
]
```

---

## map/apis.json

One entry per discovered API surface. Write `[]` if the project exposes no APIs.

```json
[
  {
    "id": "<kebab-case-api-id>",
    "type": "<rest|graphql|grpc|event|cli>",
    "path": "<source file or spec file, e.g. src/api/routes.ts>",
    "summary": "<one sentence: what this API does>",
    "_meta": {
      "generated_at": "<ISO 8601 timestamp>",
      "ttl_days": 7,
      "derived_from": ["<glob pattern, e.g. src/api/**>"],
      "fingerprint": "<sha256:hash>",
      "stale": false,
      "stale_reason": null
    }
  }
]
```

---

## map/_provenance.json

Master provenance index for all derived map files. One entry per file tracked.

```json
{
  "modules": {
    "generated_at": "<ISO 8601 timestamp>",
    "ttl_days": 7,
    "derived_from": ["<glob pattern covering all module roots>"],
    "fingerprint": "<sha256:hash>",
    "stale": false,
    "stale_reason": null
  },
  "apis": {
    "generated_at": "<ISO 8601 timestamp>",
    "ttl_days": 7,
    "derived_from": ["<glob pattern covering API source files>"],
    "fingerprint": "<sha256:hash>",
    "stale": false,
    "stale_reason": null
  }
}
```

---

## tasks/refresh-map.task.md

```markdown
# Task: Refresh Map

## id
refresh-map

## Goal
Re-derive all `map/` files from current project source and update `_provenance.json`.

## Context Priority
1. ales.manifest.json
2. map/_provenance.json

## Steps
1. Load `map/_provenance.json` and identify all tracked map files.
2. For each tracked file, check if the source files in `derived_from` have changed since `generated_at` by comparing fingerprints.
3. For each changed or TTL-elapsed file, re-derive the map content from current source using the same logic as the original bootstrap.
4. Update the file with new content.
5. Update `map/_provenance.json`: set `generated_at` = now, recompute `fingerprint`, set `stale: false`, clear `stale_reason`.
6. Update `ales.manifest.json`: set `updated_at` = now, set `repo_sha` = current git SHA.
7. Report which files were refreshed and which were unchanged.

## Expected Output
All `map/*.json` files are up to date with current source. `_provenance.json` shows `stale: false` for all entries.

## Stop Conditions
- Source files in `derived_from` cannot be read — stop and report.
- Map file cannot be written — stop and report.
```

---

## tasks/check-staleness.task.md

```markdown
# Task: Check Staleness

## id
check-staleness

## Goal
Detect and report any stale or TTL-expired entries in `map/_provenance.json` without modifying any files.

## Context Priority
1. ales.manifest.json
2. map/_provenance.json

## Steps
1. Load `map/_provenance.json`.
2. For each tracked entry:
   a. If `stale: true` — mark as STALE.
   b. If `generated_at` + `ttl_days` < now — mark as TTL_EXPIRED.
   c. Otherwise — mark as FRESH.
3. Compute fingerprint of current source files in each entry's `derived_from` and compare to stored fingerprint.
   - Mismatch → mark as STALE with reason "source fingerprint changed".
4. Report results. Do not write any files.

## Expected Output
A staleness report listing each tracked map file with status: FRESH, STALE, or TTL_EXPIRED, and the reason for any non-fresh status.

## Stop Conditions
- `map/_provenance.json` does not exist — stop and suggest running `ales-apply` (update mode).
```
