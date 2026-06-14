# Token Efficiency Playbook

Shared guidance for all studios. Apply these rules for every agent session.

---

## 1. RTK — wrap every terminal command

RTK filters and compresses output, saving 60–90% tokens on log/build-heavy commands.

```bash
rtk git status
rtk git log -10
rtk pnpm test
rtk pnpm build
```

Run `rtk gain` to see cumulative savings. Never use raw `git`, `pnpm`, or `docker` for output-heavy commands.

---

## 2. Use index files — never scan folders

| Need | Read this |
|---|---|
| Which skills exist? | `agent-context/map/skills-index.md` |
| Which tasks exist? | `agent-context/map/tasks-index.md` |
| Where is content? | `agent-context/map/folders.md` |
| Which workflow applies? | `agent-context/map/workflow.md` |
| Reusable patterns? | `agent-context/patterns/index.md` (AnimationStudio) |

Never run `ls`, `Get-ChildItem`, or `find` on `agent-context/` to discover capabilities.

---

## 3. Start with minimal context — escalate only on confirmed gaps

Every active studio has `agent-context/intent/context-profiles.md`. Use it.

| Profile | When | Typical cost |
|---|---|---|
| `minimal` | First message, status check, quick continuation | ~3 files / ~400 tokens |
| `task` | Task type is known | ~6–8 files / ~1 500 tokens |
| `full` | Audit, framework debug, new skill generation | all agent-context / ~5 000+ tokens |

**Default to `minimal`. Escalate only when you hit a confirmed knowledge gap.**

---

## 4. Prefer summaries and registries over raw sources

- ResearchStudio: use `wiki/synthesis.md` instead of re-reading individual corpus files.
- All studios: use `skills-index.md` before opening any individual skill file.
- All studios: use `tasks-index.md` before listing or reading the tasks/ folder.
- Vault projects: read `map.md` (project status) before scanning subfolders.

---

## 5. Follow abbreviated load-order for common workflows

Every active studio has `agent-context/map/load-order.md` listing exact file sequences for:

1. Resume existing project/curriculum
2. Start new project/curriculum
3. Continue an active task
4. Export / handoff

Use those sequences verbatim. Do not build a broader context read from scratch.

---

## 6. Respect manifest read bounds

```yaml
bounds:
  max_files_per_query: 10   # never exceed per query
  max_lines_per_file: 150   # stop at this line
  max_depth: 2              # do not recurse deeper
```

Bounds declared in `agent-context/plugins/hub/manifest.md` apply to Hub queries **and** to agent self-reads for status/summary work.

---

## 7. Load anti-goals-brief — not the full anti-goals doc

Each studio has `agent-context/intent/anti-goals-brief.md` — a 5-bullet summary. Load it at session start in `minimal` mode. Load the full `anti-goals.md` only when a refusal decision is ambiguous.

---

## 8. Resolve vault.md aliases once per session

The `vault.md` alias file is stable. Read it once, resolve the paths you need (`{projects}`, `{curricula}`, etc.), and reuse those resolved paths for the rest of the session.

---

## 9. Check context freshness before reusing derived files

Before acting on any file that was derived from another source (map.md, workflow.md, synthesis.md, skills-index.md), check:

1. Does it have a `_meta` block with `ttl_days` and `updated`?
2. Is `today - updated > ttl_days`? → Mark stale, call `ALES/skills/reference/verify-context-freshness.skill.md`.

Do not skip this for synthesis docs older than their TTL. Stale context is a hidden token cost: it forces correction rounds later.

---

## 10. Declare a token budget contract before deep tasks

Tasks with more than 6 files in their `Load:` block should declare a budget at the top:

```yaml
token_budget:
  context_profile: task
  max_context_files: 8
  escalation_trigger: "gap in task spec or skills"
  max_output_tokens: 2000
```

If a task's load would exceed the declared budget, surface the trade-off before proceeding.

---

## References

- `ALES/ales.manifest.json` — repo-wide registry and governance metadata
- `ALES/skills/reference/verify-context-freshness.skill.md` — staleness check skill
- `Vault/studios/ResearchStudio/projects/007-agent-context-management/output/report.md` — compaction strategy matrix
- `Vault/Knowledge/concepts/kc-cost-control-requires-budgeting-prompt-and-output.md` — cost-control doctrine
