# ALES v2 — Design Plan

**Status:** Active  
**Stage:** POC + Research Paper  
**Supersedes:** `../V1/planV1.md`  
**Inputs:** `../V1/planV1.md`, `../V1/v1-review-suggestions.md`, `../V1/generalization.md`

---

## Why V2 Exists

V1 had the right core idea. The four-layer taxonomy, the staleness model, the "experienced engineer" mental model, and the focus on bounded context are all worth keeping.

What V1 got wrong:

- It overclaimed. Deterministic traversal and reproducible outputs were stated as properties of ALES when they are only achievable if the loader, runtime, prompts, tools, and environment are also deterministic. ALES enables these properties; it does not guarantee them.
- It underspecified. The execution contract needed schemas, trace formats, source-of-truth rules, and a minimal validation concept before any claim about behavior could be tested.
- It conflated research claims with production claims. At this stage, ALES is a POC and a research paper idea, not an enterprise platform. The V1 language drifted toward production readiness without earning it.
- The skill and task distinction was directionally right but not sharp enough to guide implementation.

V2 keeps the idea and makes it defensible.

---

## Scope

V2 is a research and POC document.

The goal is to:

1. Define ALES precisely enough to build a reference implementation and a case study.
2. Make the claims testable against a real software-repo POC.
3. Support a position paper arguing that the four-layer taxonomy solves a real and underaddressed problem in LLM agent workflows.

V2 is **not** a product specification, Azure implementation plan, enterprise security architecture, or market positioning document. Those are later stages.

---

## Core Thesis (Preserved From V1)

> ALES is a repository-side structured knowledge and execution contract that lets any capable LLM agent complete repo-bound tasks with bounded context, auditable traversal, and measurable outputs — complementing whatever agent runtime, IDE, or tool the agent uses.

Three target properties:

| Property | POC-stage definition |
|---|---|
| **Bounded context** | The agent loads only the files required for the current task. Context size is measurable in tokens before and after ALES is applied. |
| **Auditable traversal** | Given the same task and repo state, the agent always loads files in the same declared order. The trace records which files were loaded and why. |
| **Model-portable** | ALES is a repo-side spec, not a runtime library. Any agent that follows the contract should be able to use the same context structure, regardless of model or tooling. |

**What changed from V1:**

- "Deterministic traversal" → "Auditable traversal." Determinism is a property of the full system, not of ALES alone. ALES provides structure and traceability; determinism still requires the loader and runtime to be deterministic.
- "Reproducible outputs" → dropped as a V2 claim. Output reproducibility depends on model, prompt, temperature, runtime state, and tool behavior. ALES can make inputs more consistent, which improves repeatability, but this needs empirical evidence before being stated as a property.
- "Model-agnostic" → "Model-portable." Agnostic implies identical behavior across models, which is false. Portable is the accurate claim: the repo contract works without being tied to one vendor.

---

## Positioning

ALES is a complement to existing agent infrastructure, not a replacement for any part of it.

| What ALES is NOT | What ALES IS |
|---|---|
| Not an agent runtime | The repo-side knowledge layer that runtimes consume |
| Not a model or prompt library | A project-specific context contract for any model |
| Not a RAG system | A structured index of project-specific intent, maps, and workflows |
| Not an IDE plugin | A folder convention and contract that IDEs can optionally surface |
| Not an orchestration framework | A task and skill taxonomy that any orchestrator can route through |
| Not an MCP tool | A repo convention that MCP tools, agents, and CLIs can read from |

The skill remains stable even if two different agents use different APIs or tools to execute it. ALES defines what the workflow is. The runtime decides how to execute it.

---

## Revised Four-Layer Taxonomy

The layers are unchanged. The descriptions are tighter.

| Layer | Question it answers | Owner | Freshness model | V2 note |
|---|---|---|---|---|
| **`intent/`** | *Why* is the system shaped this way? | Human (agent may draft) | Manual; rarely stale | Ground for human design decisions. Agents may propose changes; humans approve. |
| **`map/`** | *Where* do things live and *what* exists? | Agent (derived from source) | Fingerprint + TTL → batch refresh | Derived from authoritative source artifacts. Not a substitute for OpenAPI, migrations, or types. |
| **`skills/`** | *How do you do X in this specific project?* | Human (agent may suggest) | Versioned; stale when intent changes | Captures project-specific workflows, constraints, and conventions. Runtime-tool-agnostic by design. |
| **`tasks/`** | *How does an agent execute a generic class of work?* | Spec-level; shared | Spec-versioned; portable across repos | Generic procedure. Invokes skills for project-specific behavior. |

**Two Truths, revised:**

| Truth | Role | Conflict rule |
|---|---|---|
| **Code / authoritative artifacts** | Ground truth for what the system does | Always wins over docs and intent |
| **Intent** | Human-owned context for why the system is shaped this way | Never contradicts code on factual claims; describes rationale, constraints, goals |

**Source-of-truth precedence (default for software repos):**

| Source | Precedence |
|---|---|
| Tests, runtime behavior, API schemas, DB migrations | Highest — what the system actually does |
| Production code | What the system is built to do |
| ADRs, architecture docs | Why those choices were made |
| README, docs | How to work with the system |
| Agent-authored maps | Derived summaries; lowest precedence |

Repos may override this order in `ales.manifest.json`.

---

## Task vs. Skill Semantics

This distinction needs to be precise because it determines what ALES authors and what runtime authors.

**Task**

A task is a generic, goal-oriented procedure that is portable across any project using ALES.

A task defines:
- The goal class (e.g., "add a feature", "debug a test failure", "refresh the map")
- The sequence of steps at a high level
- The context loading priorities
- The expected output schema
- When to stop and ask

A task does **not** define which tools, APIs, or agent capabilities to use. That is the runtime's job.

**Skill**

A skill is a project-specific recipe for completing a workflow in this repo.

A skill defines:
1. What context to load first
2. What ordered steps to follow
3. What project-specific constraints apply
4. What evidence must be present before the step is complete
5. How to verify the output
6. When to stop and ask a human

A skill does **not** specify vendor-specific tool calls. If Agent A uses an IDE file tool and Agent B uses a CLI, both can follow the same skill.

**Example:**

Skill: `add-endpoint.skill.md` in a .NET repo might say:

```
1. Load the relevant controller and application service.
2. Add the endpoint method following the existing pattern in this repo.
3. Add or update the contract DTO.
4. Add a unit test and an integration test following the test conventions.
5. Verify the OpenAPI surface matches the intended shape.
6. Run the test command declared in the manifest.
```

That is stable. It does not matter whether Copilot, Claude Code, or a custom CLI agent executes it.

---

## Minimal POC Contract

For the POC, we need the smallest set of files and schemas that prove the contract works.

### Required files

```
/agent-context/
├── ales.manifest.json      ← spec version, repo SHA, source-of-truth policy, task registry
├── intent/
│   ├── overview.md         ← system purpose, key decisions, non-goals
│   └── conventions.md      ← naming, patterns, constraints human agents must follow
├── map/
│   ├── modules.json        ← module/service listing with provenance metadata
│   └── apis.json           ← API surface summary (derived; lower precedence than OpenAPI spec)
├── skills/
│   └── *.skill.md          ← one skill per recurring workflow type
└── tasks/
    └── *.task.json         ← one task per generic work class
```

### Minimum schema surface (V2 POC)

| Schema | Minimum required fields |
|---|---|
| **Manifest** | `ales_version`, `repo_sha`, `source_of_truth_order`, `task_registry[]`, `schema_version` |
| **Task** | `id`, `goal`, `steps[]`, `context_priority[]`, `expected_output_schema`, `stop_conditions[]` |
| **Skill** | `id`, `applies_to_task`, `steps[]`, `constraints[]`, `verification[]`, `ask_when[]` |
| **Map entry** | `id`, `type`, `path`, `_meta` (see below) |
| **Provenance `_meta`** | `generated_at`, `ttl_days`, `derived_from[]`, `fingerprint`, `stale`, `stale_reason` |
| **Trace** | `task_id`, `skill_id`, `files_loaded[]`, `files_skipped[]`, `stale_warnings[]`, `token_estimate`, `verification_result` |

Full JSON schema files are a follow-up artifact (`schema-roadmap.md`). These fields define the minimum that the POC validator must check.

---

## Staleness And Provenance Model

Preserved from V1. Refined for clarity.

Every derived file in `map/` carries a `_meta` block:

```json
{
  "_meta": {
    "generated_at": "2026-05-01T10:00:00Z",
    "ttl_days": 7,
    "derived_from": ["src/Orders/**"],
    "fingerprint": "sha256:...",
    "stale": false,
    "stale_reason": null
  }
}
```

**Two phases:**

1. **Detection — continuous, cheap**  
   Any agent, mid-task, may call `mark_stale` when it observes drift. One write. No refresh yet.

2. **Refresh — deliberate, batch**  
   Triggered explicitly. Sweeps for `stale == true`, expired TTL, or fingerprint mismatch.  
   - `map/` entries: agent re-derives autonomously.  
   - `intent/` entries: agent drafts a diff, requires human approval before merge.

**V2 correction from V1:**

Provenance at the file level is required in V2. Full claim-level provenance (tracking which specific sentence in `intent/architecture.md` was derived from which function in which file) is a research goal, not a POC requirement.

---

## Execution Loop V2

```
1. RESOLVE     → match user request to a task in the task registry
2. PLAN        → load task; select applicable skill; estimate token budget; check for stale context
3. LOAD        → load only high-priority context files declared by the task and skill
4. EXECUTE     → run steps in declared order
   ├─ on missing context  → escalate priority; load next tier
   ├─ on stale context    → emit stale warning; continue or stop depending on severity
   ├─ on ambiguous flow   → STOP; emit ask
   └─ on budget exceeded  → emit partial result + reason
5. VERIFY      → check output against expected_output_schema in the task definition
6. EMIT        → return result + trace (task_id, skill_id, files loaded, token estimate, verification result)
```

**Guaranteed behaviors:**
- No silent context expansion beyond declared priorities.
- Trace is mandatory on every task execution.
- Stale context must be reported, not silently used.
- Agents must stop and ask rather than silently resolve ambiguous state.

**Action vocabulary (V2):**

| Action | Meaning |
|---|---|
| `load` | Read file(s) into context |
| `search` | Scoped search within the repo |
| `identify_flow` | Match task to flow in `map/flows/` |
| `follow_flow` | Traverse a declared flow |
| `inspect_code` | Read symbols within loaded files |
| `invoke_skill` | Apply a project-specific skill |
| `mark_stale` | Tag a context file as stale |
| `emit` | Produce structured output with trace |
| `ask` | Request clarification; terminates the current step |

---

## Evaluation Protocol (For The Paper)

The paper needs something measurable. This is the minimum viable evaluation for a position paper with case study.

**Setup:**
- One real software repo (the POC repo, TBD).
- One or two representative tasks (e.g., "add a small endpoint", "debug a failing test").
- Three conditions:
  1. No ALES context (agent has only the raw repo).
  2. Single instruction file (e.g., `agents.md` or `copilot-instructions.md`).
  3. ALES structured context (task + skill + bounded map + trace).

**Measurements:**

| Metric | What it tests |
|---|---|
| Files loaded | Bounded context: is ALES narrower than no-context? |
| Token estimate | Cost: is ALES cheaper than full-dump? |
| Task success | Quality: does the output satisfy the task goal? |
| Clarification turns | Ambiguity: does ALES reduce unnecessary questions? |
| Test/build pass | Correctness: does the code work? |
| Trace completeness | Auditability: is the execution path inspectable? |
| Stale warnings | Provenance: does the agent surface drift correctly? |

**Baseline comparison table for the paper:**

| Approach | Bounded context | Provenance | Task portability | Human intent layer | Freshness model |
|---|---|---|---|---|---|
| No context | ✗ | ✗ | ✗ | ✗ | ✗ |
| agents.md / CLAUDE.md | Partial | ✗ | ✗ | Partial | ✗ |
| RAG only | Partial | Partial | ✗ | ✗ | Partial |
| ADRs only | ✗ | ✗ | ✗ | Partial | ✗ |
| ALES | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## Domain Generalization

The V1 generalization insight is preserved as a thesis for the paper, not a V2 POC requirement.

The four-layer taxonomy applies across any domain where a human has tacit knowledge an agent lacks and where outputs are derived from structured inputs.

| ALES Layer | Code Repo | Stats Presentation | Animation |
|---|---|---|---|
| `intent/` | Architecture, design decisions | Audience, narrative arc | Storyboard, emotional beats |
| `map/` | Modules, APIs, data flows | Datasets, charts, slide sections | Scenes, assets, timelines |
| `skills/` | How to add a feature *here* | Brand palette, citation style | Easing curves, naming conventions |
| `tasks/` | debug, add-feature, refresh | add-slide, update-data-source | add-scene, sync-audio, export |

**V2 scope boundary:** The POC validates the taxonomy for software repositories. The generalization claim belongs in the paper as a hypothesis, supported by a real-world ALES implementation as a secondary illustrative example. It is not a production promise.

---

## Security Considerations For The POC

Not enterprise security. Minimum awareness for a credible research artifact.

`tasks/` and `skills/` are privileged instruction surfaces. A change to a skill can steer agent behavior across every repo that uses that skill.

For the POC, the minimum required:

- Document this risk explicitly in the spec.
- Recommend that `tasks/` and `skills/` changes require explicit human review before being treated as trusted.
- Note that CODEOWNERS or equivalent is the production control; the POC assumes a trusted single-author environment.
- Do not embed secrets or credentials in any `agent-context/` file.

---

## Explicitly Deferred To Later Stages

These are real concerns. They are named here so they are not forgotten, but they are not V2 or POC blockers.

| Deferred feature | When it becomes relevant |
|---|---|
| Full RBAC and least-privilege controls | Enterprise adoption |
| Audit logging and compliance | Regulated environments |
| Signed or trusted context files | Multi-author or adversarial environments |
| Tenant and project isolation | Hosted multi-org deployment |
| Observability dashboards | Production monitoring |
| Full claim-level provenance | V3 research target |
| Distributed orchestration | Multi-agent V2+ |
| Non-code production validation | Post-paper expansion |
| Cross-org skill/task marketplace | Product stage |
| Azure or cloud deployment | Product stage |

---

## Open Questions For The POC

1. Which real software repo will serve as the POC target?
2. Which one or two tasks will be the benchmark tasks?
3. What is the minimum schema set needed before the reference loader can be built?
4. Should `ales validate` be a CLI script, a GitHub Action, or both?
5. How will the baseline conditions (no-ALES, single-file) be controlled for fair comparison?
6. Should the paper present only one task or compare two to show generalization across task types?
7. What is the approval UX for intent drafts in the POC: `_pending/` folder, CLI prompt, or PR?

---

## Next Artifacts

In order:

1. `planV2.md` — this file.
2. `schema-roadmap.md` — draft minimal JSON schemas for manifest, task, skill, map entry, provenance, and trace.
3. `evaluation-protocol.md` — detailed benchmark setup, conditions, and metrics.
4. Update `paper-plan.md` — align paper framing with V2 revised claims and evaluation design.
5. Reference loader — ~200 LOC implementation against the POC repo.
6. POC run — execute the two benchmark tasks under three conditions, record traces.
