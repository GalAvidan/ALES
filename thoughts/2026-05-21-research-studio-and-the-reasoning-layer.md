# Research Studio & the AI Reasoning Layer — Conversation Handoff

**Date:** 2026-05-21
**Status:** Thinking / pre-design
**Author:** Working session between user and Copilot
**Purpose:** Self-contained record of a strategy conversation so it can be resumed in a fresh chat (e.g., GitHub.com Copilot) without loss of context.

> **For the next agent reading this:** This is a handoff. The user has not committed to any code changes yet. The deliverables below are *proposals discussed*, not decisions made. Pick up by confirming priorities with the user before scaffolding anything.

---

## 0. Repository Context

The user works across several projects in `c:\Git\`. Two are central to this conversation:

- **`ALES/`** — *Agent-Layer Execution Specification*. A specification (with V1–V4 design plans and a paper plan) for a structured, repository-embedded knowledge contract that lets any capable LLM agent operate with **bounded context, deterministic traversal, and model-agnostic execution**. ALES defines a `/agent-context/` folder with content types (`intent/`, `map/`, `skills/`, `tasks/`, `traces/`) and, in V4, a Scope Stack (ecosystem / enterprise / workspace / repo / project).
- **`AnimationStudio/`** — A working ALES instance and the paper's case study #1. A monorepo (pnpm) with adapter contracts, spec types, a Motion Canvas adapter, and several animation projects (`why-ice-floats`, `why-sky-is-blue`, `character-pilot`). Its `agent-context/` folder is the lived implementation of ALES.

Other projects mentioned but not central: `PlanOrchestratorAI/`, `Skool/`.

**Key prior insight from the user:** the `AnimationStudio/agent-context/` folder structure was originally derived from ALES, not invented for animation. ALES is the methodology; AnimationStudio is one instance of it.

---

## 1. The Conversation in Four Acts

### Act 1 — "Am I building the wrong layer?"

The user opened by questioning whether they had been over-investing in the **animation pipeline** (scene composition, rendering, adapters) when AI video generators (Sora, Veo, Runway) ship constantly. They suspected the real value was in an **AI reasoning layer** that turns plain-English intent into working animation scripts.

**Position taken in the conversation:**

- The pipeline was **not a mistake**, but treating it as the *product* would be.
- Diffusion video models generate **pixels**; AnimationStudio generates **editable, deterministic, re-renderable scenes**. That difference is the moat.
- LLMs are far better at producing structured artifacts (specs, JSON, code) than coherent long-form video. The pipeline is the *substrate the AI can write into*.
- Recommendation: **freeze pipeline scope**, treat it as infrastructure / compile target, and elevate the reasoning layer to first-class status.

**Defined the reasoning layer concretely:**

```
raw intent (chat / doc / voice)
   → [Intent Extraction]      structured brief (audience, goal, tone, length, beats)
   → [Pedagogical Planning]   ordered explanation graph (concepts → deps → analogies)
   → [Script Synthesis]       script.md (existing template)
   → [Spec Synthesis]         spec.md (existing template)
   → [Code Generation]        scene/composition TS via adapter contract
   → [Critique & Revision]    self-review of script/spec/render with diff proposals
```

The reasoning layer **owns** intent → brief → script → spec → adapter input plus revision loops. It **does not own** rendering, primitives, encoding, asset management — those stay in the pipeline. The contract between them is the existing **`spec-types`** package and the **adapter contract**. This is essentially compiler architecture: frontend (reasoning) → IR (spec) → backend (adapter) → artifact (video).

**Risks identified:**

1. *"It's just prompts."* — Defended by investing in artifacts and feedback loops, not prompt cleverness.
2. *Quality cliff / generic outputs.* — Defended by genuine pedagogical reasoning (concept dependency graphs, analogy selection).
3. *Big labs add structured-output → video.* — Mitigated by the adapter abstraction; the user can adopt their backend.
4. *Scope explosion.* — Mitigated by picking a vertical: **explanatory / educational animation** (3–10 min science explainers), already evidenced by existing projects.

**Defensibility comes from:** editability, pedagogical reasoning, determinism + version-controlled specs, and a compounding domain corpus of vetted explanations and visual idioms.

### Act 2 — "Can I make this a portable methodology?"

The user asked whether they could create a skill/skill-set encoding an "Interpretable Context Methodology" applicable to any project — for example, doing deep research on an academic study, then later feeding that research into AnimationStudio for explanation.

**Initial framing (before realising ALES already existed):** the conversation proposed extracting AnimationStudio's `agent-context/` taxonomy as a portable kernel called "Interpretable Context Architecture" with five layers (intent / map / skills / tasks / templates), three properties (layered intent, compile-target separation, task atomicity), and a discipline rule: every task must cite the intent/map/glossary entries it depends on, otherwise the structure becomes a museum.

A Research Studio mapping was sketched:

| AnimationStudio | ResearchStudio (proposed) |
|---|---|
| `intent/overview.md` | research question, hypothesis, scope |
| `intent/anti-goals.md` | out-of-scope subfields, claims won't make |
| `intent/glossary.md` | domain terms with sources |
| `intent/decisions/` | methodology ADRs (inclusion criteria, source trust) |
| `map/folders.md` / `map/workflow.md` | corpus map, research loop |
| `skills/` | literature-search, claim-extraction, citation-verification, contradiction-detection |
| `tasks/` | ingest-paper, extract-claims, build-claim-graph, find-counterevidence, write-section, peer-review-self |
| `templates/spec.template.md` | **`research-spec.template.md`** — the structured IR of the research |
| `templates/script.template.md` | `paper-summary.template.md`, `claim-card.template.md`, `lit-review.template.md` |
| `adapter-contract` | output adapters: markdown report, slide deck, **AnimationStudio spec** ← the bridge |

**Key insight:** `research-spec.md` is to a research project what `spec.md` is to an animation. It's the editable, machine-writable, human-readable IR. Once both studios speak in specs, research becomes a first-class input to animation, not a manual handoff.

**Bridge pipeline imagined:**

```
raw curiosity
  → ResearchStudio (intent → corpus → claim graph → research-spec)
  → "explain this" task
  → AnimationStudio (research-spec → script → animation-spec → render)
  → editable explainer video
```

### Act 3 — "Look at ALES too — that's where the agent-context folder came from."

The user pointed out that AnimationStudio's structure originated in **ALES**, which already has:

- A four-layer content taxonomy (`intent/`, `map/`, `skills/`, `tasks/`) plus `traces/` in V3
- A provenance & staleness model (lazy detection, batch refresh, fingerprint-based)
- A closed action vocabulary and execution loop with mandatory traces
- **V4's Scope Stack** — scope (ecosystem / enterprise / workspace / repo / project) is orthogonal to content type
- A paper plan (`Docs/Plans/paper-plan.md`) that already lists Stats Presentation as an unfilled third-domain column in its generalization table

This reframed everything. The user has not been *inventing* a methodology — they already specified it, named it, and wrote four design iterations and a paper plan. The "Interpretable Context Architecture" name from Act 2 is unnecessary; **ALES is the answer.** AnimationStudio is case study #1.

### Act 4 — "Then what does Research Studio mean *in ALES terms*?"

Re-anchored conclusion of the conversation:

- **A Research Studio is not a new framework. It is a second ALES instance — the missing case study #2 the paper needs.**
- **V4's Scope Stack earns its keep with two studios:** with one project you can't distinguish the scope stack from a folder convention; with two, the **workspace tier becomes load-bearing** because the cross-studio contract lives there.
- **Traces are the killer feature for research.** Every "paper says X" emission carries a trace to which source files were loaded, which extraction skill ran, and what the fingerprint was. That's the audit trail academic work demands. ALES already has this — research is a domain that *requires* it, which is itself a paper-worthy validation.
- **Provenance does what it was designed for, in a domain it wasn't designed for.** When a source paper is updated or retracted, every downstream claim becomes stale automatically via the existing V3 staleness model. Strong external validation of the model.

**Proposed Scope Stack layout for the workspace:**

```
agent-context/                          # workspace scope
  intent/
    cross-studio-conventions.md         # shared glossary, shared decisions
  tasks/
    research-spec-to-animation-script.task.md   # the bridge task
projects/
  research-studio/agent-context/        # project scope
  animation-studio/agent-context/       # project scope (already exists)
```

The bridge task lives at workspace scope, imports `research-spec.json` from the research project's `map/`, applies a narrative-arc skill, and emits an animation `spec.md` into the animation project's input. **Provenance is preserved end-to-end:** every visual beat traces back to a claim, which traces back to a paper section.

This provenance chain through structured knowledge is the thing no current AI video tool can match — not because of rendering, because of the structured-knowledge chain itself.

**Research-specific anti-goal flagged:** unlike a wrong easing curve, a wrong claim is catastrophic. `intent/anti-goals.md` for ResearchStudio must include *"the agent never asserts an unverified claim"* and the `extract-claims` skill must hard-fail rather than fabricate. ALES's closed action vocabulary already has `ask` for exactly this purpose; the discipline must be enforced at skill-definition time.

---

## 2. Key Decisions and Stances Reached

These are positions the conversation converged on, not yet committed to code:

1. **Animation pipeline stays.** It is the substrate, not the product. Freeze its scope; resist new features unless the reasoning layer demands them.
2. **Reasoning layer becomes first-class.** Likely a new package (e.g., `packages/reasoning/`) with sub-modules: `intent`, `planning`, `script-gen`, `spec-gen`, `critique`. Currently implicit in `agent-context/tasks/*.md` — fine as prototype, not as product.
3. **Vertical: explanatory animation** (3–10 min science explainers). Existing projects already prove this wedge.
4. **Critique loop early.** Generated → critiqued → revised → re-rendered, all from one prompt. This is what separates demo from product.
5. **Research Studio = ALES case study #2.** Not a new framework.
6. **Bridge task lives at workspace scope.** It is the artifact that justifies V4's Scope Stack and threads provenance from paper → claim → spec → animation.
7. **`research-spec` is a `map/` artifact** — derived, provenance-tracked, staleable. Not a freeform doc.
8. **Paper §6 (Domain Generalization) gains a real second column.** Two case studies in different domains turns "domain-agnostic" from assertion to demonstration.

---

## 3. Open Questions for the Next Session

The next chat should pick up with these unresolved threads:

1. **Should Research Studio live inside `ALES/` (as `case-study/research/`) or as a sibling project at `c:\Git\ResearchStudio\`?** Affects the workspace-scope `agent-context/` location.
2. **Where does the workspace-scope `agent-context/` actually live in a multi-repo world?** ALES V4 defines workspace scope; the practical filesystem location across separate git repos is not yet decided.
3. **What is the v0 schema of `research-spec`?** Sketch in the conversation:
    ```yaml
    question: ...
    claims:
      - id: C1
        statement: ...
        evidence: [paper-A§3, paper-B§7]
        confidence: high
        counter-evidence: [paper-C]
    contradictions: [...]
    open-questions: [...]
    narrative-arc: [C1 → C3 → C7 → conclusion]
    ```
    Needs formalising as a real schema with provenance fields per ALES conventions.
4. **What is the thin-slice end-to-end demo?** Proposal: one paper → one `research-spec` → one 2-minute animated explainer. Which paper?
5. **Does the reasoning layer get refactored out of `agent-context/tasks/` into a TypeScript package, or does it stay agent-context-only?** Trade-off: code package gives types and tests; agent-context-only stays consistent with ALES philosophy that tasks are agent procedures, not application code.
6. **Empirical validation experiments for the paper.** Mentioned but not designed: ALES vs. no-context vs. full-dump on a real task, measuring token cost and output consistency. Could the Research Studio build double as the experimental harness?
7. **Conflict resolution** between Animation conventions and Research conventions when they disagree at the workspace scope (e.g., glossary terms with different meanings).

---

## 4. Concrete Proposed Next Steps

In priority order, if the user chooses to proceed:

1. **Draft `case-study/research/` skeleton** under ALES with at minimum:
   - `intent/overview.md` (research question framing, anti-goal: never assert unverified claims)
   - `intent/anti-goals.md`
   - `intent/glossary.md` (seeded with ALES terms used in research context)
   - `templates/research-spec.template.md` (the IR)
   - `templates/claim-card.template.md`
   - Stub `tasks/`: `ingest-paper`, `extract-claims`, `build-claim-graph`
2. **Draft the workspace-scope bridge task** `research-spec-to-animation-script.task.md`. This exposes contract gaps that single-studio work cannot.
3. **Pick a thin-slice paper** and walk it through the pipeline manually before automating, capturing what worked and what the agent had to ask about.
4. **Update `Docs/Plans/paper-plan.md`** to elevate Research Studio to case study #2 and Stats Presentation to case study #3 (or future work).
5. **Decide on the reasoning-layer package question** (open question 5 above) before writing TypeScript.

The user's explicit request at the end of the conversation was *not* to scaffold any of this yet — it was to capture the conversation for later. Confirm before creating files in either ALES or AnimationStudio.

---

## 5. Glossary (for the next agent)

- **ALES** — Agent-Layer Execution Specification. The user's own specification for repository-embedded agent context.
- **Scope Stack (V4)** — Ecosystem / Enterprise / Workspace / Repo / Project. Orthogonal to content type.
- **Content types** — `intent/` (why), `map/` (where/what), `skills/` (how-here), `tasks/` (how-generic), `traces/` (memory).
- **Provenance** — every derived artifact records `derived_from`, `fingerprint`, `ttl_days`, `stale`. Source change → downstream stale.
- **Reasoning layer** — the AI frontend that turns plain-English intent into structured specs the pipeline can compile. Not yet a real package.
- **Pipeline layer** — AnimationStudio's existing infrastructure: spec types, adapter contract, Motion Canvas adapter, render pipeline. Treat as compile target.
- **Bridge task** — a workspace-scope ALES task that converts a `research-spec` into an animation `spec.md`, threading provenance through.
- **Thin slice** — one paper → one research-spec → one short animation, end-to-end, manually first.

---

## 6. Resume Prompt for the Next Chat

Copy-paste this into the next session if useful:

> I'm resuming a strategy conversation about ALES and AnimationStudio. Read `ALES/thoughts/2026-05-21-research-studio-and-the-reasoning-layer.md` for full context. Specifically I want to continue with **[pick one: open question N / proposed next step N / a new angle]**. Do not scaffold files until I confirm.

---

*End of handoff.*
