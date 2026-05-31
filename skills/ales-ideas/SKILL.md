---
name: ales-ideas
description: >
  Manages the canonical ideation flow for ALES ecosystems using Vault
  `agent-context/.ideas/` as the pre-plan workspace.
  Supports creating ideas, updating exploratory drafts, and promoting
  ready ideas into `agent-context/.plans/` with traceable backlinks.
  Use when: capturing early concepts, refining open questions, migrating
  freeform thoughts into structured idea files, or promoting ideas to plans.
---

# ALES Ideas

This skill standardizes ideation before implementation planning.

## Canonical location

- Ideas live in: `Vault/agent-context/.ideas/`
- Plans live in: `Vault/agent-context/.plans/`
- `ALES/thoughts/` is archive-only and should not receive new ideation entries.

## Operations

### 1) Create idea

Use when no matching idea exists.

1. Load `Vault/agent-context/.ideas/index.md`.
2. Choose a stable folder slug under `.ideas/`.
3. Create `index.md` and `idea_00.md` from template.
4. Fill frontmatter and the five required sections:
   - What
   - Why
   - Open Questions
   - Explorations
   - Related
5. Register the entry in `.ideas/index.md`.

### 2) Update idea

Use when an existing idea already covers the topic.

1. Append new findings to `Explorations` in the latest `idea_NN.md`.
2. Update `Open Questions` and `status` (`seed` or `exploring`).
3. Add backlinks to related plans, tasks, decisions, and source notes.
4. If content diverges significantly, create `idea_(N+1).md` as a new snapshot.

### 3) Promote idea to plan

Use only when the idea is implementation-ready.

Promotion criteria:

1. Goal and scope are clear.
2. Open questions are resolved or explicitly deferred.
3. Execution phases can be sequenced.

Steps:

1. Create target folder under `Vault/agent-context/.plans/<plan-slug>/`.
2. Generate plan files (`index.md`, `plan_00.md`, additional files as needed).
3. Set idea `status: promoted` and `promoted-to` backlink.
4. Keep idea content intact for provenance and historical context.

## Safety rules

- Do not skip the idea stage for exploratory requests.
- Do not mark an idea as `ready` without explicit evidence in the file.
- Do not delete migrated thought notes; keep them as references within idea folders.
