# Wiki LLM Plan

## Purpose

This plan describes the Wiki LLM idea as a standalone workflow pattern for building personal knowledge bases with LLMs.

It does not discuss ALES integration (see `ales-absorption-plan.md` for that).

---

## Core Idea

Wiki LLM is a pattern where the LLM incrementally builds and maintains a persistent, interlinked wiki from raw sources.

The key distinction from RAG:

> RAG retrieves and synthesizes at query time. Wiki LLM compiles knowledge into a persistent artifact that compounds over time.

The human curates sources and asks questions. The LLM does all the maintenance: summarizing, cross-referencing, filing, updating, and flagging contradictions.

---

## Architecture

### Layer 1: Raw Sources

Immutable. The human curates them. The LLM reads but never modifies.

Examples:

- articles, papers, reports
- book chapters
- transcripts (meetings, podcasts, interviews)
- notes, journal entries
- images, data files
- web clippings

### Layer 2: Generated Wiki

LLM-maintained markdown files. The LLM owns this layer entirely.

Page types:

- **Source summaries** — one per ingested source
- **Concept pages** — one per important idea or topic
- **Entity pages** — one per person, organization, place, or thing
- **Synthesis pages** — cross-source analyses
- **Comparison pages** — structured side-by-side evaluations
- **Contradiction pages** — unresolved conflicts between sources
- **Open questions** — what the wiki cannot yet answer

### Layer 3: Operating Schema

A configuration document that tells the LLM:

- how the wiki is structured
- what conventions to follow
- what workflows to use for ingest, query, and maintenance

This could be a single `AGENTS.md` file, or a full `agent-context/` structure.

---

## Core Operations

### Ingest

Process a new source into the wiki.

Steps:

1. Read the source.
2. Extract key claims, entities, concepts, and evidence.
3. Create or update a source summary page.
4. Update all affected concept and entity pages.
5. Detect contradictions with existing claims.
6. Update `index.md`.
7. Append to `log.md`.

Important: ingest updates the whole affected knowledge graph, not just a single summary page. A single source may touch 10–15 wiki pages.

### Query

Answer a question against the wiki.

Steps:

1. Read `index.md` to find relevant pages.
2. Read relevant wiki pages.
3. If needed, check raw sources for verification.
4. Synthesize an answer with citations.
5. Decide whether to file the answer as a new wiki page.

Filing rule: file the answer when it represents durable, reusable knowledge (a comparison, synthesis, timeline, or unresolved question). Do not file purely conversational or low-confidence answers.

### Lint

Health-check the wiki periodically.

Checks:

- orphan pages (no inbound links)
- missing backlinks
- outdated summaries superseded by newer sources
- contradicted claims not yet flagged
- important concepts mentioned but lacking their own page
- pages without citations
- duplicate or near-duplicate pages
- stale open questions

Output: a health report with suggested fixes.

---

## Key Files

### `wiki/index.md`

Content-oriented catalog of everything in the wiki.

Each entry:

- link to the page
- one-line summary
- optional source count
- optional last-updated date

Organized by category (sources, concepts, entities, syntheses, comparisons, contradictions, open questions).

The LLM reads this first before answering most questions. At moderate scale (~100 sources, ~hundreds of pages), the index alone can replace embedding-based retrieval.

### `wiki/log.md`

Chronological, append-only record of operations.

Format:

```markdown
## [2026-05-21] ingest | Source Title

- Added: source-summary, concept-page-X
- Updated: entity-page-Y, synthesis-Z
- Contradictions: claim A vs claim B
- Open questions: ...
```

The consistent prefix format makes the log parseable with simple tools.

### `wiki/contradictions.md`

Central tracker for unresolved conflicts between sources.

Each entry:

- the contradicted claim
- source A's position
- source B's position
- current status (unresolved / resolved / superseded)
- what evidence would resolve it

### `wiki/open-questions.md`

Questions the wiki cannot yet answer. Serves as a research backlog guiding future source collection.

---

## Folder Structure

```text
project/
├── raw/
│   ├── sources/
│   └── assets/
└── wiki/
    ├── index.md
    ├── log.md
    ├── overview.md
    ├── open-questions.md
    ├── contradictions.md
    ├── concepts/
    ├── entities/
    ├── sources/
    ├── syntheses/
    └── comparisons/
```

---

## Human Role

The human:

- curates and selects sources
- decides what matters and what to explore
- reviews important syntheses
- resolves ambiguous interpretations
- asks the questions that drive exploration
- approves major structural changes

The human does not need to:

- maintain cross-links manually
- remember every contradiction
- re-summarize old sources
- update every affected page after each ingest

## LLM Role

The LLM:

- ingests and processes sources
- creates and updates all wiki pages
- maintains cross-references and backlinks
- flags contradictions
- files useful query results as new pages
- maintains index and log
- cites sources in every claim
- asks the human when evidence is insufficient

The LLM does not:

- modify raw sources
- invent claims without evidence
- hide uncertainty
- silently resolve contradictions
- overwrite human decisions without approval

---

## Use Cases

- **Research** — deep-dive on a topic over weeks; papers, articles, reports accumulating into a structured wiki with an evolving thesis
- **Reading a book** — chapter-by-chapter wiki with characters, themes, plot threads, connections
- **Personal** — goals, health, self-improvement; journal entries and articles building a structured self-picture
- **Business** — internal wiki fed by Slack, meetings, customer calls; stays current because the LLM does the maintenance
- **Competitive analysis** — tracking competitors over time with contradiction detection when claims change

---

## Practical Tips

- **Obsidian** as the browsing tool (graph view shows wiki shape, orphans, hubs)
- **Git** for version history and collaboration
- **Web Clipper** (Obsidian extension) for converting articles to markdown
- **One source at a time** for engaged ingestion; batch for less critical material
- **Marp** for generating slide decks from wiki content
- **Dataview** (Obsidian plugin) for queries over page frontmatter

---

## Scaling Considerations

At small scale (~50 sources): `index.md` is sufficient for navigation.

At medium scale (~100–300 sources): consider adding a local search tool (e.g., qmd, or a simple BM25 script).

At large scale (500+ sources): proper search infrastructure becomes necessary; the index file is no longer enough for the LLM to find relevant pages efficiently.
