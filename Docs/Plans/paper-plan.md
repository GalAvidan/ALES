# ALES Academic Paper — Plan

## Core Thesis

> LLM agents introduced a new knowledge management problem in software development.
> Raw model capability is necessary but insufficient — without structured, repository-embedded
> knowledge contracts, agents either operate with insufficient context (producing generic, inconsistent
> results) or receive excessive context (wasting tokens and degrading performance).
>
> ALES proposes that this is a **specification problem, not a model problem**.
> Its four-layer taxonomy (`intent/`, `map/`, `skills/`, `tasks/`) with provenance-aware staleness
> enables bounded context, deterministic traversal, and model-agnostic execution.

---

## The "New Layer" Framing

AI agents create a **third truth plane** in software:

| Plane | What it answers | Who owns it |
|---|---|---|
| **Code** | What the system does | Machine / developer |
| **Documentation** | What the system means | Developer |
| **Agent-Context** | How to *work with* the system | Human + agent, co-authored |

The first two planes have existed since software began.
The third plane is new — brought into existence by LLM agents with bounded context windows.
ALES is a specification for the third plane.

The key insight: without direction, it doesn't matter how capable the agent is.
Vast model knowledge without structured repository contracts produces expensive, inconsistent, and
poorly-targeted results. Proper structure multiplies the agent's capability; absent structure, that
capability is dispersed.

---

## Paper Type

**Position paper with a case study** (recommended first step).
No experiments required. Publishable as an arXiv preprint or workshop paper.

Open question: Do we run empirical experiments in a later version (e.g., ALES vs. no-context vs.
full-dump on a real task, measuring token cost and output quality)?

---

## Target Venue (TBD)

Options:
- LLM4Code / NL4SE workshops at ICSE or FSE
- SE for AI track
- arXiv preprint (design/position paper)
- Technical blog post as a stepping stone

---

## Paper Structure

### 1. Abstract
- State the problem (context pollution / context starvation)
- State the thesis (specification problem, not a model problem)
- Name the contributions
- Name the case study

### 2. Introduction
- Open with the practitioner problem: agents fail on real repos, not because they lack knowledge of the world, but because they lack structured knowledge of *this* project
- Motivate the "new layer" framing
- State contributions clearly

### 3. Background and Related Work

| Approach | How ALES differs |
|---|---|
| AGENTS.md / CLAUDE.md / Cursor rules | Ad hoc; no taxonomy; no provenance; model-specific |
| RAG systems | Retrieval-time, not design-time; no human taxonomy; no staleness model |
| MemGPT / Mem0 | External / runtime memory; model-dependent; not repo-embedded |
| ADR (Architecture Decision Records) | Only one layer (intent/decisions); no map, skills, or tasks |
| LangChain / LlamaIndex | Frameworks, not specifications; vendor lock-in |
| OpenAPI specs | Only API surface; no intent or workflow |

### 4. The ALES Framework

#### 4.1 The Four-Layer Taxonomy

| Layer | Answers | Owner | Freshness |
|---|---|---|---|
| `intent/` | *Why* is the system shaped this way? | Human (agent may draft) | Manual; rarely stale |
| `map/` | *Where* do things live, *what* exists? | Agent (derived) | TTL + stale-tag → batch refresh |
| `skills/` | *How do you do X in THIS project?* | Human (agent may suggest) | Only when intent changes |
| `tasks/` | *How does an agent execute a class of work?* | Spec / shared | Spec-versioned; portable |

Key distinctions to formalize:
- A **task** is a generic procedure, portable across repos
- A **skill** is a project-specific recipe (assumes the agent has general capability)
- A task may *invoke* a skill

#### 4.2 The Staleness Model

> Staleness is detected continuously and lazily, but resolved deliberately and in batch.

Two phases:
1. **Detection** — any agent, mid-task, marks an entry stale in one write. No refresh yet.
2. **Refresh** — triggered explicitly (user, CI, or a `refresh` task). Sweeps stale entries, re-derives `map/` autonomously, drafts diffs to `intent/` for human approval.

Provenance header on every derived file:
```json
{
  "_meta": {
    "generated_at": "...",
    "ttl_days": 7,
    "stale": false,
    "derived_from": ["src/Orders/**"],
    "fingerprint": "sha256:..."
  }
}
```

#### 4.3 The Execution Loop

```
1. RESOLVE     → match user request to a task definition
2. PLAN        → load task; compute step DAG; estimate token budget
3. LOAD-HIGH   → load only high-priority context
4. EXECUTE     → run steps in order
   ├─ on missing context → escalate priority
   ├─ on ambiguous flow  → STOP, ask (no guessing)
   └─ on budget exceeded → emit partial + reason
5. VERIFY      → check output against expected_output schema
6. EMIT        → return result + trace (files loaded, steps run, tokens used)
```

Mandatory guarantees:
- No silent context expansion beyond declared priorities
- Trace is mandatory (makes "deterministic" verifiable)
- Budgets declared per task

#### 4.4 Closed Action Vocabulary (v1)

```
load            → read file(s) into context
search          → scoped search
identify_flow   → match task to flow in map/flows/
follow_flow     → traverse a flow's path
inspect_code    → read symbols within loaded files
invoke_skill    → apply a project-specific skill
mark_stale      → tag a context file as stale
emit            → produce structured output
ask             → request user clarification (terminates step)
```

### 5. Case Study: AnimationStudio

AnimationStudio is a working ALES implementation: a Remotion-based animation workflow where
every agent action is routed through `/agent-context/`.

Walk through one concrete task (e.g., "build a new animation scene from a spec") showing:
- Which files the agent loads (bounded context, not the whole repo)
- How the task file routes to skills
- How provenance tracks which spec a scene is derived from
- What the trace looks like

Metrics to consider capturing:
- Token budget for the task vs. loading the full repo
- Number of files loaded vs. files in the repo
- Output consistency across model runs

### 6. Domain Generalization

ALES is domain-agnostic. The same four planes apply to:

| ALES Layer | Code Repo | Stats Presentation | Animation |
|---|---|---|---|
| `intent/` | Architecture, design decisions | Audience, narrative arc | Storyboard, emotional beats |
| `map/` | Modules, APIs, data flows | Datasets, charts, slide sections | Scenes, assets, timelines |
| `skills/` | How to add a feature *here* | Brand palette, citation style | Easing curves, naming conventions |
| `tasks/` | debug, add-feature, refresh | add-slide, update-data-source | add-scene, sync-audio, export |

Generalization rule to state formally:
> Any project where a human has tacit knowledge an agent lacks, and where outputs are derived
> from structured inputs, benefits from the ALES layer system.

### 7. Open Problems and Limitations

- **Entry-point discovery** — how does an agent reliably find `/agent-context/` without being told?
- **CI integration** — how does the staleness refresh hook into automated pipelines?
- **Conflict resolution** — what happens when code and `intent/` contradict each other?
- **Approval UX for intent drafts** — critical for creative/non-code domains where map entries require human judgment
- **Empirical validation** — the three core properties (bounded, deterministic, model-agnostic) are testable but untested

### 8. Conclusion and Research Agenda

- Restate the thesis and contributions
- Name the empirical experiments that would validate the claims
- Identify what v2 would add

---

## Claims Requiring Literature Support

Every claim below needs a matching entry in `knowledge-base.md`.

| Claim | Search terms / known sources |
|---|---|
| Context window quality degrades with size and noise | "Lost in the Middle" (Liu et al., 2023) |
| LLM agents produce inconsistent results on real codebases | SWE-bench (Jimenez et al., 2024) |
| Structured prompts improve agent output quality | Chain-of-thought, prompt engineering literature |
| ADR as a pattern for capturing design rationale | Nygard (2011), ADR literature |
| RAG limitations for agent context management | RAG survey papers |
| External memory system limitations (MemGPT, Mem0) | Packer et al. (2023), Mem0 paper |
| Separation of concerns as a software engineering principle | Dijkstra (1982), classic SE literature |
| Token cost as a measurable quality dimension | OpenAI pricing studies, context window research |

---

## Files to Create

| File | Purpose | Status |
|---|---|---|
| `Docs/Plans/paper-plan.md` | This file — high-level plan, structure, claims | ✅ Created |
| `Docs/References/knowledge-base.md` | Curated papers, tools, concepts for claim verification | ⬜ To do |
| `Docs/Paper/01-introduction.md` | Introduction section draft | ⬜ To do |
| `Docs/Paper/02-related-work.md` | Related work section draft | ⬜ To do |
| `Docs/Paper/03-framework.md` | ALES framework section draft | ⬜ To do |
| `Docs/Paper/04-case-study.md` | AnimationStudio case study draft | ⬜ To do |
| `Docs/Paper/05-generalization.md` | Domain generalization section draft | ⬜ To do |
| `Docs/Paper/06-discussion.md` | Open problems and limitations | ⬜ To do |
| `Docs/Paper/07-conclusion.md` | Conclusion and research agenda | ⬜ To do |

---

## Open Decisions

1. **Paper type** — Position paper with case study, or empirical study?
2. **Target venue** — ICSE/FSE workshop, arXiv preprint, or blog post?
3. **Case study scope** — AnimationStudio only, or add a second case study (e.g., PlanOrchestratorAI) to demonstrate cross-domain generalization?
4. **Empirical component** — Run experiments comparing ALES vs. baselines, or stay theoretical/observational for v1?
