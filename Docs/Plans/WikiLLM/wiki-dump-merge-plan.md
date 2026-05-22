# ALES Merge Plan — Wiki Dump Ingestion

## Purpose

Define a new merge path where ALES adopts a **dump-first** knowledge workflow:

1. scan project knowledge files
2. dump them into a structured source area
3. let the agent consume that dump
4. generate and maintain ALES structure from it

This plan is additive and does not replace existing Wiki LLM plans.

---

## Core Idea

Use markdown (and later other text artifacts) as the initial durable input layer.

Instead of hand-authoring all ALES context first, the system builds it from a project knowledge dump.

```text
project scan -> knowledge dump -> agent ingest -> /agent-context refresh
```

---

## Scope

### In scope

- markdown-first project scan
- deterministic dump folder and manifest
- ingest flow that maps dump content into ALES layers
- incremental refresh when dump content changes

### Out of scope (for this plan)

- replacing current ALES plans
- non-text parsing quality (PDF/images/OCR)
- embedding/vector infrastructure

---

## Proposed Flow

### 1) Scan

Collect candidate knowledge files from configured paths (for v1: `**/*.md` with include/exclude rules).

Output: list of discovered files + fingerprints.

### 2) Dump

Materialize a normalized dump snapshot:

```text
knowledge-dump/
├── manifest.json
├── sources/
│   └── ...mirrored markdown files...
└── batches/
    └── 2026-05-22T12-00-00Z.json
```

`manifest.json` tracks:

- file path
- hash/fingerprint
- discovered_at
- source type
- include/exclude reason

### 3) Ingest

Run an ingest task that consumes `knowledge-dump/` and derives:

- `agent-context/intent/*` drafts (themes, goals, assumptions)
- `agent-context/map/*` entities/modules/relations
- `agent-context/skills/*` candidate project-specific recipes
- provenance links back to dump sources

### 4) Refresh

On each new scan:

- unchanged files: skip
- changed/new files: re-ingest impacted areas
- removed files: mark derived artifacts stale

---

## ALES Mapping Rules (v1)

1. Every derived ALES artifact must include `derived_from` entries that point to dump source files.
2. Dump content is immutable per batch; refresh creates a new batch record.
3. Conflicts found across dump files are recorded as explicit contradiction artifacts.
4. Human-authored intent remains review-gated; ingest may draft but not silently overwrite.

---

## Delivery Phases

### Phase 0 — Spec Draft

- define dump folder contract
- define manifest schema
- define ingest task contract

### Phase 1 — Markdown Pipeline

- implement markdown scanner
- implement dump writer
- implement ingest-to-`map/` and draft `intent/`

### Phase 2 — Incremental Maintenance

- changed-file detection
- partial re-ingest
- stale/contradiction reporting

---

## Success Criteria

- Running scan + ingest on a repo with markdown docs produces a valid `agent-context/` skeleton.
- Re-running after a markdown change updates only impacted derived artifacts.
- Agent can answer project questions using generated ALES context with traceable provenance.

---

## Open Questions

- Should dump live inside repo (`knowledge-dump/`) or be generated under `.ales/`?
- Should excluded files be logged in manifest for auditability?
- What is the minimal contradiction schema for v1?
- Which ingest outputs require human approval before becoming canonical?
