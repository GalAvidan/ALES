# ALES Absorption Plan — What to Change in the Spec

## Purpose

This plan defines what ALES needs to add or change to support knowledge-base workflows inspired by Wiki LLM.

It does not describe Wiki LLM itself (see `wiki-llm-plan.md` for that).

It focuses on concrete spec-level additions to ALES.

---

## Decision: What Goes Into Core ALES vs. a Domain Profile

### Absorb into core ALES

These concepts are general enough to benefit any ALES domain, not just knowledge bases:

1. **Knowledge-product distinction** — ALES should formally distinguish between agent-context artifacts (optimized for task execution) and generated knowledge products (optimized for human reading). Both are governed by provenance, but they serve different audiences.

2. **Durable query outputs** — when an agent produces a meaningful synthesis during task execution, ALES should support filing it as a new derived artifact with provenance. This prevents valuable work from disappearing into chat history.

3. **Contradiction as a first-class staleness event** — ALES already tracks staleness (source changed → derived artifact may be outdated). Contradiction is the symmetric case: two sources disagree. ALES should add contradiction as a provenance event type alongside staleness.

4. **Human-readable index/log as optional companions** — `index.md` as a human companion to `map/`, and `log.md` as a human companion to `traces/`. Optional, not required.

### Keep in a domain profile (not core)

These are useful for knowledge-base projects but should not be forced on code repos or other domains:

- Obsidian/git workflow assumptions
- Fixed wiki folder structure
- Personal-knowledge-base-only framing
- Markdown-only output format assumptions
- Specific page types (concept pages, entity pages, etc.)

---

## Spec Addition 1: Knowledge-Product Layer

ALES currently defines what agents consume. It should also define what agents produce as durable outputs.

**Proposed concept:**

A **knowledge product** is an agent-generated artifact that:

- lives outside `agent-context/` (it is not execution context)
- is governed by `agent-context/` (tasks and skills control how it is maintained)
- carries provenance metadata (derived_from, generated_at, fingerprint)
- is optimized for human consumption

**Relationship to existing layers:**

```text
agent-context/        ← execution contract (existing)
output/               ← knowledge products (new concept)
```

The exact folder name (`output/`, `wiki/`, `generated/`) is left to the domain profile. The spec addition is the concept and its provenance rules.

---

## Spec Addition 2: Filing Policy for Task Outputs

ALES tasks currently emit results with a trace. The spec should add an optional `file_output` step in the execution loop:

```text
6. EMIT        → return result + trace
7. FILE        → (optional) persist result as a new derived artifact with provenance
```

**Filing criteria (suggested defaults, overridable per profile):**

File when the output is:

- a synthesis across multiple sources
- a comparison or analysis
- a decision or resolution
- an unresolved question worth tracking
- reusable beyond the current conversation

Do not file when the output is:

- purely conversational
- low-confidence
- already superseded by an existing artifact
- not backed by traceable sources

---

## Spec Addition 3: Contradiction Events

ALES provenance currently tracks:

- `stale: true/false`
- `stale_reason`
- `derived_from`
- `fingerprint`

Add a new provenance event type:

```json
{
  "type": "contradiction",
  "claim": "Orders use eventual consistency",
  "source_a": "intent/architecture.md",
  "source_b": "raw/sources/new-paper.md",
  "detected_at": "2026-05-22T09:00:00Z",
  "status": "unresolved"
}
```

Contradictions should:

- be surfaced during ingest and lint operations
- be tracked in a dedicated artifact (e.g., `map/contradictions.json`)
- trigger review, not silent resolution

---

## Spec Addition 4: Human-Readable Companions

Two optional files that any ALES project can add:

| File | Purpose | Companion to |
|---|---|---|
| `index.md` | Content-oriented catalog of all knowledge artifacts | `map/` |
| `log.md` | Chronological record of agent operations | `traces/` |

These are not required by the spec. They are recommended when the project has human readers who browse the knowledge base directly (e.g., in Obsidian, VS Code, or GitHub).

---

## Spec Addition 5: Ingest/Query/Lint Task Family

ALES should define three generic task templates for knowledge-management domains:

| Task | Purpose |
|---|---|
| `ingest-source.task` | Process a new source into the knowledge base |
| `query-knowledge-base.task` | Answer a question against accumulated knowledge |
| `lint-knowledge-base.task` | Check health and consistency of the knowledge base |

These are generic tasks (portable across projects). Domain-specific skills define how each project performs them.

---

## New ALES Concept: Domain Profile

A **domain profile** is a documented set of conventions for applying ALES to a specific domain.

A profile specifies:

- which optional spec features to use
- folder layout conventions
- required tasks and skills
- provenance granularity expectations
- output artifact types

The first profile should be the **Knowledge Wiki Profile**, which defines how to apply ALES to an LLM-maintained knowledge base.

---

## Recommended Next Steps

1. Draft the Knowledge Wiki Profile as a formal ALES domain profile document.
2. Draft the three generic knowledge-base task specs.
3. Decide the provenance format for contradiction events.
4. Update `paper-plan.md` to reference Wiki LLM as a non-code case study.
5. Use Research Studio as the first concrete implementation of the Knowledge Wiki Profile.
