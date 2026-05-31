# ALES — Agent-Layer Execution Specification

> *Give any LLM agent the structured knowledge it needs to work in your project — without flooding it with your entire codebase.*

---

## The Grand Idea

LLM agents are powerful, but they have a fundamental problem when working on real projects: they either know too little (and make generic, inconsistent decisions) or are given too much context (wasting the token budget and becoming slower and less reliable).

**ALES solves this by establishing a structured, machine-readable knowledge contract that lives inside your repository** — an `/agent-context/` folder that tells any capable agent *exactly* what it needs to know, organized by how often that knowledge changes and how it is used.

The result: **bounded context, deterministic traversal, and reproducible outputs — independent of the underlying model.**

---

## The Three Core Properties

| Property | What it means |
|---|---|
| **Bounded context** | The agent loads only the files required for the current task. Context size is measurable and predictable. |
| **Deterministic traversal** | Given the same task and the same repo state, the agent always loads files in the same order and follows the same steps. |
| **Model-agnostic** | ALES is a specification, not a library. Any capable LLM that follows the contract produces consistent results. |

---

## The Four-Layer Taxonomy

ALES organizes project knowledge into four layers, each answering a different question:

| Layer | Question it answers | Who owns it | How often it changes |
|---|---|---|---|
| **`intent/`** | *Why* is the system shaped this way? | Human (agent may draft) | Rarely — architecture and goals change slowly |
| **`map/`** | *Where* do things live and *what* exists? | Agent (derived from source) | TTL-based; refreshed when source changes |
| **`skills/`** | *How do you do X specifically in this project?* | Human (agent may suggest) | Only when intent or conventions change |
| **`tasks/`** | *How does an agent execute a generic class of work?* | Spec / shared | Spec-versioned; portable across repos |

**Key distinction:** A *task* is a generic procedure (e.g., "add a feature", "refresh a chart") that is portable across any project. A *skill* is a project-specific recipe (e.g., "how we apply brand colors in *this* deck") that captures the institutional knowledge usually locked in a human's head.

---

## Folder Structure

```
/agent-context/
├── ales.manifest.json         ← spec version, schemas, repo SHA
│
├── intent/                    ← authored, slow-changing
│   ├── overview.md
│   ├── architecture.md
│   ├── conventions.md
│   ├── glossary.md
│   └── decisions/             ← ADR-style "why we chose X"
│
├── map/                       ← derived, agent-maintained
│   ├── modules.json
│   ├── apis.json
│   ├── flows/
│   └── _provenance.json
│
├── skills/                    ← project-specific recipes
│   └── *.skill.md
│
└── tasks/                     ← agent execution procedures
    └── *.task.json
```

---

## Provenance — The Defining Property

Every file in `/agent-context/` knows which source files it was derived from. When those source files change, the context entry automatically becomes stale.

```json
{
  "_meta": {
    "generated_at": "2026-04-15T10:00:00Z",
    "ttl_days": 7,
    "stale": false,
    "derived_from": ["src/Orders/**"],
    "fingerprint": "sha256:..."
  }
}
```

Staleness is **detected continuously and lazily** (any agent mid-task can mark an entry stale in one write), but **resolved deliberately and in batch** (a dedicated refresh pass sweeps stale entries, re-derives map files autonomously, and drafts updates to intent files for human approval).

---

## It's Not Just for Code

The four-layer taxonomy is domain-agnostic. Any project where a human has tacit knowledge an agent lacks — and where outputs are derived from structured inputs — benefits from this contract.

| ALES Layer | Code Repo | Stats Presentation | Animation |
|---|---|---|---|
| **`intent/`** | System architecture, design decisions | Narrative arc, audience rules | Storyboard intent, emotional beats |
| **`map/`** | Modules, APIs, data flows | Slides ↔ datasets | Scenes ↔ assets ↔ audio |
| **`skills/`** | How to add a feature in *this* repo | Brand palette, number format | Easing curves, color grading |
| **`tasks/`** | debug, add-feature, refresh | add-slide, refresh-chart | add-scene, export-render |

---

## Ideation Standard

Early concepts should be captured in `Vault/agent-context/.ideas/` as structured idea files.

- `.ideas/` is the canonical pre-plan workspace.
- Promote ideas to `Vault/agent-context/.plans/` only when they are implementation-ready.
- `ALES/thoughts/` remains archive-only for historical notes.

See `skills/ales-ideas/SKILL.md` for the create/update/promote workflow.

---

## How an Agent Executes a Task

```
1. RESOLVE     → match user request to a task definition
2. PLAN        → load task; compute step DAG; estimate token budget
3. LOAD-HIGH   → load only high-priority context files
4. EXECUTE     → run steps in declared order
   ├─ on missing context → escalate priority, load more
   ├─ on ambiguity       → STOP and ask (no silent guessing)
   └─ on budget exceeded → emit partial result + reason
5. VERIFY      → check output against expected_output schema
6. EMIT        → return result + trace (files loaded, steps run, tokens used)
```

The trace is mandatory — it makes "deterministic" verifiable by inspection.

---

## Examples

- [`Docs/Examples/stats-presentation/`](Docs/Examples/stats-presentation/) — Q3 board deck with live data sources
- [`Docs/Examples/animation/`](Docs/Examples/animation/) — 30-second brand spot animation

---

## Design Documents

- [`Docs/Plans/V1/planV1.md`](Docs/Plans/V1/planV1.md) — Core thesis, execution loop, and open questions for v1
- [`Docs/Plans/V2/generalization.md`](Docs/Plans/V2/generalization.md) — How ALES generalizes beyond code repositories
