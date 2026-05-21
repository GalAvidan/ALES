# ALES Absorption Plan for Wiki LLM Mechanics

## Purpose

This plan describes how ALES should absorb the useful mechanics from the Wiki LLM idea without collapsing ALES into a personal wiki product.

The intended outcome is:

> ALES remains the broader specification for structured, provenance-aware agent context, while Wiki LLM becomes a concrete knowledge-base profile, workflow, and case study inside the ALES ecosystem.

## Positioning

ALES should not merge with Wiki LLM as an equal replacement idea.

Instead:

- **ALES is the specification layer.**
- **Wiki LLM is a domain profile / reference workflow.**
- **Research Studio can use the Wiki LLM profile as its first non-code ALES instance.**

This preserves ALES's core identity:

- bounded context
- auditable traversal
- model portability
- provenance-aware staleness
- task and skill separation
- traceable agent execution

While absorbing Wiki LLM's strongest practical insight:

> LLM-generated knowledge should compound into durable, browsable, interlinked artifacts instead of disappearing into chat history.

## Useful Mechanics to Absorb

### 1. Human-browsable generated knowledge

ALES currently emphasizes context that agents consume.

Wiki LLM emphasizes knowledge that humans browse.

ALES should explicitly support both:

- **Agent-facing context**: structured files optimized for task execution.
- **Human-facing synthesis**: readable markdown pages, indexes, summaries, and comparisons.

This matters because non-code ALES instances, especially research, need to be useful even when no agent task is currently running.

### 2. Query results as durable artifacts

Wiki LLM has a strong rule:

> Good answers should be filed back into the wiki.

ALES should adopt this as a formal workflow option.

When an agent answers a meaningful research or synthesis question, the output should be eligible to become:

- a new synthesis page
- an updated concept page
- a comparison artifact
- an unresolved question
- a trace-linked decision note

This turns exploratory conversations into accumulated project knowledge.

### 3. Index and log files

Wiki LLM proposes two simple navigation primitives:

- `index.md` — content-oriented catalog
- `log.md` — chronological activity record

ALES already has richer concepts:

- `ales.manifest.json`
- `map/`
- `_provenance.json`
- `traces/`

But `index.md` and `log.md` are useful because they are simple, human-readable, and friendly to markdown tools like Obsidian.

ALES should add them as optional profile-level artifacts, not core required spec files.

Recommended mapping:

| Wiki LLM file | ALES role |
|---|---|
| `index.md` | Human-readable companion to `map/` |
| `log.md` | Human-readable companion to `traces/` |

### 4. Ingest / query / lint as first-class tasks

Wiki LLM's operations map cleanly into ALES tasks:

| Wiki LLM operation | ALES task |
|---|---|
| Ingest | `ingest-source.task` |
| Query | `answer-from-knowledge-base.task` |
| Lint | `lint-knowledge-base.task` |

ALES should formalize these as generic task templates for knowledge-base domains.

Project-specific skills can then define how each domain performs them.

For example:

- research paper ingestion
- book chapter ingestion
- meeting transcript ingestion
- competitive analysis ingestion
- health journal ingestion

### 5. Contradiction and stale-claim handling

Wiki LLM highlights contradictions between sources.

ALES already has the right primitives:

- provenance
- staleness
- claim-level citations
- traces
- stale refresh passes

ALES should make contradiction handling explicit in the knowledge-base profile:

- new sources may strengthen an existing claim
- new sources may weaken an existing claim
- new sources may contradict an existing claim
- contradictions should become first-class artifacts, not hidden notes

Recommended artifact:

```text
agent-context/map/contradictions.json
```

Or, for human-facing knowledge bases:

```text
wiki/contradictions.md
```

### 6. Obsidian/git workflow

Wiki LLM's "Obsidian is the IDE, LLM is the programmer, wiki is the codebase" framing is useful.

ALES should use this as a motivating explanation for non-code audiences:

- Markdown files are the source tree.
- The LLM performs edits.
- Git gives history and review.
- Obsidian gives browsing, graph view, and human inspection.

This is not required for ALES, but it is a strong reference implementation pattern.

## Proposed ALES Additions

### 1. Add a Knowledge Wiki profile

Create a documented profile:

```text
ALES Knowledge Wiki Profile
```

This profile defines how to instantiate ALES for a living knowledge base.

It should include:

- source folder conventions
- generated wiki folder conventions
- required tasks
- recommended skills
- provenance expectations
- index/log conventions
- contradiction handling
- when answers should be filed back

### 2. Add generic knowledge-base tasks

Suggested task templates:

- `ingest-source.task`
- `answer-from-knowledge-base.task`
- `lint-knowledge-base.task`
- `refresh-synthesis.task`
- `file-query-result.task`

These should live at the generic ALES task level, not inside one project.

### 3. Add knowledge-base skills

Suggested skill templates:

- `extract-claims.skill`
- `update-concept-page.skill`
- `update-entity-page.skill`
- `detect-contradictions.skill`
- `maintain-index.skill`
- `maintain-log.skill`
- `cite-source.skill`

These may start as examples in the Wiki LLM profile and later become reusable shared skills.

### 4. Add human-facing generated artifacts

ALES should distinguish between:

- **agent context artifacts** — optimized for task execution
- **knowledge product artifacts** — optimized for human reading

Suggested folders for a knowledge wiki profile:

```text
raw/                  # immutable sources
wiki/                 # LLM-maintained human-readable markdown
agent-context/        # ALES execution contract
```

The wiki is not a replacement for `agent-context/`.

The wiki is a generated product governed by `agent-context/`.

### 5. Add a filing policy for query outputs

Not every answer should become a file.

ALES should define a filing policy:

File the answer when it is:

- reusable
- a synthesis across multiple sources
- a comparison
- a decision
- a contradiction analysis
- an unresolved research question
- a stable explanation likely to be referenced later

Do not file the answer when it is:

- purely conversational
- temporary
- low-confidence
- superseded by an existing page
- not backed by citations

## Proposed Folder Shape

For the Wiki LLM profile:

```text
knowledge-wiki/
├── raw/
│   ├── sources/
│   └── assets/
├── wiki/
│   ├── index.md
│   ├── log.md
│   ├── overview.md
│   ├── concepts/
│   ├── entities/
│   ├── sources/
│   ├── syntheses/
│   └── contradictions.md
└── agent-context/
    ├── ales.manifest.json
    ├── intent/
    ├── map/
    ├── skills/
    ├── tasks/
    └── traces/
```

## Relationship to Research Studio

Research Studio should likely be the first serious implementation of this profile.

The relationship should be:

```text
ALES
└── Knowledge Wiki Profile
    └── Research Studio
```

Research Studio can specialize the generic profile with:

- stricter citation requirements
- claim-level provenance
- confidence levels
- counter-evidence tracking
- "never assert unverified claims" anti-goal
- bridge task into AnimationStudio

## Should This Become Core ALES?

Partially.

Core ALES should absorb:

- durable query outputs
- human-readable index/log as optional companions
- ingest/query/lint task family
- contradiction handling
- knowledge-product distinction

Core ALES should not absorb:

- Obsidian as a requirement
- a fixed wiki folder structure
- a personal-knowledge-base-only framing
- markdown-only assumptions for all domains

## Recommended Next Steps

1. Define the Knowledge Wiki profile as an ALES domain profile.
2. Use Research Studio as the first concrete instance.
3. Draft `ingest-source`, `answer-from-knowledge-base`, and `lint-knowledge-base` task specs.
4. Draft minimal skills for claim extraction, contradiction detection, and index maintenance.
5. Decide whether the generated wiki lives inside `agent-context/` or beside it. Recommended: beside it.
6. Update the ALES paper plan to include Wiki LLM / Research Studio as a non-code case study.

## Decision Recommendation

ALES should absorb Wiki LLM as:

> A concrete knowledge-base profile and workflow that demonstrates ALES beyond code.

It should not become:

> ALES renamed as Wiki LLM, or ALES reduced to a markdown wiki pattern.

The strongest combined framing is:

> Wiki LLM shows what a living LLM-maintained knowledge base feels like. ALES provides the specification that makes it auditable, stale-aware, task-driven, and portable across agents.
