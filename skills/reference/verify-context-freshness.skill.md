# Verify Context Freshness

Checks derived map and index files for staleness using TTL and fingerprint metadata.
Run this before re-using any derived file that has a `_meta` block.

---

## Purpose

Prevent agents from acting on stale derived context (map files, indexes, synthesis documents).
Stale context causes silent errors and forces expensive correction rounds later.

---

## When to invoke

- Before using `agent-context/map/workflow.md` if it was last updated > 7 days ago
- Before using `wiki/synthesis.md` in any ResearchStudio project
- Before using `agent-context/map/skills-index.md` or `tasks-index.md`
- Before any audit, cross-studio report, or architecture review
- When a `_meta.ttl_days` field is present in a file you are about to rely on

---

## Steps

1. **Identify derived files to check** — list files with `_meta.derived_from[]` blocks.
2. **For each derived file:**
   - Read `_meta.updated` and `_meta.ttl_days`.
   - If `today - updated > ttl_days` → mark **STALE**.
3. **For stale files:**
   - Read each entry in `_meta.derived_from[]`.
   - Check whether those source files have been modified since `_meta.updated`.
   - If any source is newer → mark **STALE+CHANGED**.
4. **Emit the freshness report** (see output format below).
5. **Act on findings:**
   - `FRESH` → proceed.
   - `STALE` (TTL expired, source unchanged) → surface warning, proceed with caution.
   - `STALE+CHANGED` → stop; call the appropriate refresh task before proceeding.

---

## Output format

```
## Freshness Report — {YYYY-MM-DD}

| File | Status | Last Updated | TTL | Source Changed |
|---|---|---|---|---|
| agent-context/map/workflow.md | FRESH | 2026-06-10 | 30d | No |
| wiki/synthesis.md | STALE+CHANGED | 2026-05-01 | 14d | Yes |
```

---

## Refresh tasks to call on STALE+CHANGED

| File type | Refresh task |
|---|---|
| `agent-context/map/` files | `agent-context/tasks/refresh-map.task.md` |
| `wiki/synthesis.md` | `agent-context/tasks/synthesize-claims.task.md` |
| `wiki/index.md` | `agent-context/skills/knowledge/update-wiki-index.skill.md` |

---

## Notes

- Agents that cannot compute exact fingerprints should use file modification date as a proxy.
- The `_meta` block format is defined in `ALES/Docs/Plans/V2/planV2.md`.
- This skill is referenced in `ALES/ales.manifest.json` under `staleness_governance`.
