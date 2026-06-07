# ADR-G01 — Creation, Review, Versioning, Deprecation, Registry

Date: 2026-06-07 | Status: Active

---

## 1. Who can create a studio

- Owner approval required.
- Must include: a purpose ADR, an `anti-goals.md`, and a `plugins/hub/manifest.md`.
- Must pass `validate-hub-contract` before merging.
- No placeholder folders. If the design is not ready, do not create the folder.

## 2. Who can create skills / tasks

- Any agent may draft.
- Merge requires the review gate (section 3 below).

## 3. Review gate checklist

- [ ] Declares full `Load:` dependencies; no circular references (skills never invoke tasks).
- [ ] No duplicate name across studios, OR explicitly single-sourced with the other as a stub (`canonical: true` in frontmatter of the authoritative copy).
- [ ] Provenance preserved (research artifacts cite `corpus/` source IDs).
- [ ] Discovery index regenerated and CI green (once AI-007 is in CI).

## 4. Versioning

- Every skill and task carries `version: N.N` in its frontmatter.
- Every change bumps the version; breaking changes bump the major.
- Hub sources continue using the existing `@vN` convention.

## 5. Deprecation

- Mark deprecated artifacts with `deprecated: true` and `replaces: path/to/new`.
- Retain for at least one sprint after all consumers are updated.
- Remove only after no task's `Load:` block references it.

## 6. Registry update step

- Creating, renaming, or removing a studio/skill/task MUST regenerate the discovery
  index (`scripts/generate-index.ps1`). CI enforces no drift.
