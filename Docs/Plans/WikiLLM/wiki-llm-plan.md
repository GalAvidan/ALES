# Wiki LLM Plan

## Purpose

This plan captures the Wiki LLM idea as a concrete workflow that can be compared with, implemented through, and eventually used as a case study for ALES.

The goal is not to replace ALES.

The goal is to define a practical, usable version of Wiki LLM that ALES can govern.

## Core Idea

Wiki LLM is a pattern for building a living knowledge base where:

- raw sources are curated by a human
- the LLM reads and integrates sources
- the LLM maintains a persistent markdown wiki
- useful query results are filed back into the wiki
- contradictions, updates, and open questions are tracked over time
- the wiki becomes richer with every source and every serious question

The core distinction from RAG:

> RAG retrieves and synthesizes at query time. Wiki LLM compiles knowledge into a persistent artifact that compounds over time.

## Main Layers

### 1. Raw sources

Raw sources are immutable.

They are the source of truth.

Examples:

- articles
- papers
- transcripts
- notes
- book chapters
- images
- datasets
- web clippings

The LLM may read these files but should not modify them.

### 2. Generated wiki

The wiki is the LLM-maintained knowledge product.

It contains:

- source summaries
- concept pages
- entity pages
- synthesis pages
- comparison pages
- contradiction pages
- open questions
- overview pages
- index and log files

Humans read and browse this layer.

The LLM owns maintenance.

### 3. Operating schema

The schema tells the LLM how to maintain the wiki.

In an ALES implementation, this should be represented by:

- `agent-context/intent/`
- `agent-context/map/`
- `agent-context/skills/`
- `agent-context/tasks/`
- `agent-context/traces/`

In a simpler non-ALES implementation, this might be a single `AGENTS.md`, `CLAUDE.md`, or similar instruction file.

## Recommended Folder Structure

```text
wiki-llm/
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
│   ├── comparisons/
│   └── contradictions.md
└── agent-context/
    ├── ales.manifest.json
    ├── intent/
    ├── map/
    ├── skills/
    ├── tasks/
    └── traces/
```

## Core Operations

### 1. Ingest

Ingest processes a new source into the wiki.

Expected flow:

1. Read the new source.
2. Identify key claims, entities, concepts, and evidence.
3. Create or update a source summary page.
4. Update relevant concept and entity pages.
5. Detect contradictions with existing pages.
6. Update `index.md`.
7. Append an entry to `log.md`.
8. Write a trace if using ALES.

Important rule:

> Ingest should update the whole affected knowledge graph, not only create a summary page.

### 2. Query

Query answers a question against the wiki.

Expected flow:

1. Read `wiki/index.md`.
2. Identify relevant wiki pages.
3. Read relevant source summaries and synthesis pages.
4. If needed, inspect raw sources for verification.
5. Answer with citations.
6. Decide whether the answer should be filed back into the wiki.

Useful query outputs that should often be filed:

- comparisons
- synthesis pages
- timelines
- contradiction analyses
- decision notes
- research questions
- explainers

### 3. Lint

Lint checks the health of the wiki.

Expected checks:

- orphan pages
- missing backlinks
- outdated summaries
- contradicted claims
- pages without citations
- important concepts without pages
- duplicate pages
- weak index entries
- stale open questions
- source pages not reflected in synthesis pages

Lint should produce:

- a short health report
- suggested fixes
- optional direct updates if the user approves

## Key Files

### `wiki/index.md`

Purpose:

- content-oriented catalog
- first file the LLM reads before answering most questions
- helps humans browse the wiki

Recommended sections:

- overview
- sources
- concepts
- entities
- syntheses
- comparisons
- contradictions
- open questions

Each entry should include:

- link
- one-line summary
- optional source count
- optional last-updated date

### `wiki/log.md`

Purpose:

- chronological history of what happened
- useful for humans and agents resuming work

Recommended entry format:

```markdown
## [2026-05-21] ingest | Source Title

- Added: ...
- Updated: ...
- Contradictions: ...
- Open questions: ...
```

The consistent prefix makes the log searchable with simple tools.

### `wiki/contradictions.md`

Purpose:

- central place to track unresolved conflicts
- prevents contradictions from being hidden in summaries

Each contradiction should include:

- claim
- source A
- source B
- status
- current interpretation
- what evidence would resolve it

### `wiki/open-questions.md`

Recommended optional file.

Purpose:

- capture questions the wiki cannot yet answer
- guide future source collection
- make ignorance explicit

This file should become a research backlog.

## ALES Mapping

| Wiki LLM concept | ALES concept |
|---|---|
| Raw sources | Authoritative source artifacts |
| Wiki pages | Human-facing generated knowledge product |
| Source summaries | Derived map/synthesis artifacts |
| Schema | ALES manifest + tasks + skills |
| Ingest | Generic task |
| Query | Generic task |
| Lint | Generic task |
| index.md | Human-readable map companion |
| log.md | Human-readable trace companion |
| Contradictions | Provenance/staleness/claim conflict artifacts |

## Initial ALES Task Set

Suggested tasks:

- `ingest-source.task`
- `answer-from-wiki.task`
- `lint-wiki.task`
- `refresh-page.task`
- `file-answer.task`

## Initial ALES Skill Set

Suggested skills:

- `summarize-source.skill`
- `extract-claims.skill`
- `update-concept-page.skill`
- `update-entity-page.skill`
- `detect-contradictions.skill`
- `maintain-index.skill`
- `maintain-log.skill`
- `cite-sources.skill`

## Human Role

The human should:

- choose sources
- decide what matters
- review important synthesis
- resolve ambiguous interpretations
- approve major schema changes
- ask the questions that drive exploration

The human should not need to:

- manually maintain cross-links
- remember every contradiction
- repeatedly summarize old sources
- update every affected page after each ingest

## LLM Role

The LLM should:

- ingest sources
- maintain pages
- update links
- flag contradictions
- file useful answers
- maintain index and log
- cite sources
- ask when evidence is insufficient

The LLM should not:

- modify raw sources
- invent claims
- hide uncertainty
- silently resolve contradictions
- overwrite human-owned intent without approval

## Relationship to Research Studio

Research Studio can be a stricter version of Wiki LLM.

Compared to a general personal wiki, Research Studio should add:

- claim-level provenance
- confidence levels
- counter-evidence tracking
- citation requirements
- source quality ranking
- explicit anti-goal: never assert unverified claims
- bridge to AnimationStudio through `research-spec`

In this framing:

```text
Wiki LLM = general pattern
ALES Knowledge Wiki Profile = formalized version
Research Studio = rigorous research implementation
```

## Suggested Additional Files

Not all should be created immediately.

Recommended later files:

1. `knowledge-wiki-profile.md`
   - Formal ALES profile for Wiki LLM-style projects.

2. `research-studio-plan.md`
   - Concrete plan for turning the profile into ALES's research case study.

3. `task-specs.md`
   - Drafts for `ingest-source`, `answer-from-wiki`, and `lint-wiki`.

4. `schema-questions.md`
   - Open questions about folder layout, provenance granularity, and filing policy.

For now, the two most important files are this plan and the ALES absorption plan.

## Recommendation

Build Wiki LLM as an ALES-governed knowledge-base profile, not as a separate competing framework.

The first implementation should likely be Research Studio, because research benefits most from:

- source immutability
- claim provenance
- contradiction tracking
- durable synthesis
- linting
- traceability

This makes Wiki LLM useful on its own while strengthening ALES's claim that its taxonomy generalizes beyond code.
