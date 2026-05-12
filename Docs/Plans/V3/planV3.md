# ALES v3 — Design Plan

**Status:** Proposal / Refinement  
**Stage:** Post-POC → Empirical + Multi-Agent  
**Supersedes:** `../V2/planV2.md`  
**Inputs:** `../V2/planV2.md`, `../V1/v1-review-suggestions.md`, `../V1/generalization.md`, `../../paper-plan.md`

---

## Why V3 Exists

V2 made ALES defensible. It corrected V1's overclaims, sharpened the task/skill distinction, defined a minimal POC contract, and scoped the spec to a research artifact backed by a real software repo.

What V2 deliberately deferred — and what V3 picks up:

- **Full claim-level provenance** — V2 only required file-level `_meta`. V3 makes individual claims in `intent/` and `skills/` traceable to their source artifacts.
- **Multi-agent coordination** — V2 assumed a single-agent, single-repo world. V3 addresses how multiple agents share, update, and trust the same `/agent-context/` concurrently.
- **Empirical validation** — V2 designed the evaluation protocol. V3 runs it, reports results, and uses those results to refine the spec.
- **Adaptive context selection** — V2 declared priority tiers statically in task files. V3 introduces a lightweight learned or rule-driven selector that adjusts priority based on task history and observed load patterns.
- **Cross-repo skill/task reuse** — V2 noted a "marketplace" as a product-stage goal. V3 defines the minimum interface for a repo to import a shared skill or task from another repo or a central registry.
- **Runtime feedback loop** — V2 traces are write-once audit logs. V3 promotes them to first-class inputs: trace data feeds back into staleness scoring, context priority tuning, and skill effectiveness metrics.

V3 keeps the idea, makes it empirically grounded, and prepares it for production-readiness.

---

## Scope

V3 is an **empirical research and production-readiness document**.

The goals are:

1. Run the V2 evaluation protocol on the POC repo and report real numbers.
2. Publish a revised position paper that includes empirical results.
3. Define the minimum additions needed to move from a single-author POC to a multi-author, multi-agent repo.
4. Specify claim-level provenance precisely enough to prototype.
5. Define the cross-repo import interface for skills and tasks.

V3 is **not** yet a product release, hosted service, or enterprise security architecture. Those remain later-stage concerns.

---

## Core Thesis (Refined From V2)

> ALES is a repository-side structured knowledge and execution contract that lets any capable LLM agent complete repo-bound tasks with bounded context, auditable traversal, and measurable outputs — complementing whatever agent runtime, IDE, or tool the agent uses.

**Three preserved properties — now empirically tested:**

| Property | V3 definition |
|---|---|
| **Bounded context** | Empirically measured: ALES loads N% fewer tokens than a full-repo dump for the benchmark task set. |
| **Auditable traversal** | Trace schema v2 captures file path, load priority, load reason, and action taken per step. Traces are machine-diffable across runs. |
| **Model-portable** | Same `/agent-context/` structure used by at least two different agent runtimes (e.g., GitHub Copilot + Claude Code) in the benchmark. |

**New V3 property:**

| Property | V3 definition |
|---|---|
| **Feedback-aware** | Execution traces and staleness events feed back into context priority scores and staleness TTL calibration, making the system self-correcting over time. |

---

## What Changes From V2

| V2 | V3 |
|---|---|
| File-level provenance (`_meta` on each derived file) | + Claim-level provenance (individual assertions in `intent/` and `skills/` carry source pointers) |
| Single-agent, single-author assumed | Multi-agent concurrency model defined; executor taxonomy introduced (primary agent, spawned sub-agent, tool task, verifier) |
| Static context priority tiers in task files | Adaptive priority selector informed by trace history (memory layer) |
| Evaluation protocol designed, not run | Evaluation results reported, analyzed |
| Skills and tasks are repo-local | Cross-repo import interface defined |
| Trace is a write-once audit log | `traces/` named as the **memory layer**; trace data feeds back into staleness scores and priority calibration |
| Trusted single-author environment assumed | Multi-author review gates specified (CODEOWNERS, PR-required for `tasks/` and `skills/`) |
| No formal framing of context as shared infrastructure | Context layers explicitly framed as shared infrastructure routed into every executor's context window |

---

## Context Layers as Shared Execution Infrastructure (V3)

The screenshot below (from an independent GPT-5.4 analysis) captures the architectural framing that V3 formally adopts:

> *Context layers are the real shared infrastructure.  
> A knowledge layer, a rule layer, a task layer, maybe a memory layer —  
> all routed into the current execution context, available to whatever executor is acting:  
> primary agent, spawned sub-agent, tool task, verifier.*

This framing is precisely what ALES implements. V3 makes it explicit.

### The Four Context Sub-Layers

| Sub-layer | ALES layer | What it holds | Who owns it |
|---|---|---|---|
| **Knowledge** | `intent/` + `map/` | Why the system is shaped this way; where things live | Human (`intent/`), agent-derived (`map/`) |
| **Rule** | `skills/` | Project-specific constraints, workflows, and verification steps | Human (agent may suggest) |
| **Task** | `tasks/` | Generic execution procedures, portable across repos | Spec-level; shared |
| **Memory** | `traces/` | Execution history, priority calibration data, cross-run consistency signal | Agent-written; human-readable |

The **memory layer** is what V3's `traces/` layer provides. It is not merely an audit log — it is the accumulated execution experience of the system, read by future agents to make better context-selection decisions. Naming it explicitly as the memory layer aligns the ALES vocabulary with how multi-agent systems literature describes persistent state.

### Executor Taxonomy

Every context sub-layer is **available to all executor types**, not just the primary agent. V3 defines four executor types and their relationship to the shared context:

| Executor | Role | Context access |
|---|---|---|
| **Primary agent** | Receives the user request; drives the execution loop (RESOLVE → PLAN → LOAD → EXECUTE → EMIT) | Full read access to all four sub-layers; writes traces and `_pending/` drafts |
| **Spawned sub-agent** | Delegated a scoped sub-task by the primary agent (e.g., "refresh this map entry", "verify this output") | Reads the same `/agent-context/` snapshot as the primary agent; may write `mark_stale` and traces; may not write `_pending/` without primary agent approval |
| **Tool task** | A discrete, bounded action invoked as a step within the execution loop (e.g., `check_claims`, `search`, `inspect_code`) | Read-only context access scoped to the files declared by the invoking step; no trace write |
| **Verifier** | Checks the output of the EXECUTE phase against the `expected_output_schema` declared in the task | Read-only access to the task definition and the output under review; writes a verification result into the primary agent's trace |

**Key rule:** The context snapshot loaded at PLAN time is shared and immutable for the duration of one task execution. A spawned sub-agent or tool task may not load additional context files outside the declared priority tiers of the parent task without an explicit priority escalation emitted back to the primary agent.

**V3 concurrency implication:** The multi-agent concurrency rules already defined in this plan apply at the executor level. A spawned sub-agent that calls `mark_stale` follows the same last-write-wins-on-stale rule as any other agent. A verifier may not clear a stale flag.

---

## Revised Four-Layer Taxonomy (V3)

The layers are unchanged. The semantics are extended.

| Layer | Sub-layer name | Question it answers | Owner | Freshness model | V3 extension |
|---|---|---|---|---|---|
| **`intent/`** + **`map/`** | Knowledge | *Why* and *what* | Human / Agent-derived | Manual + fingerprint+TTL | Claim-level `_cite` pointers; trace-informed TTL calibration |
| **`skills/`** | Rule | *How to do X in this specific project?* | Human (agent may suggest) | Versioned; claim-level provenance | Import interface for shared skills |
| **`tasks/`** | Task | *How does an agent execute a generic class of work?* | Spec-level; shared | Spec-versioned; portable | Import interface; adaptive priority selector |
| **`traces/`** | Memory | *What has been done before, and what worked?* | Agent-written | Retention policy; TTL calibration | First-class read layer for adaptive selection |

---

## New Layer: `traces/` (V3)

V2 traces were outputs — written at the end of a task, useful for auditing. V3 promotes traces to a **managed layer** that the spec explicitly reads from.

```
/agent-context/
└── traces/
    ├── _index.json              ← task_id → [trace_ids], most recent first
    └── {task_id}/{run_id}.trace.json
```

**What the trace layer enables:**

1. **Adaptive priority selector** — before planning a task, the agent reads the last N traces for that task type and ranks context files by how frequently they were loaded and how useful they were (no stale warnings, verification passed).
2. **TTL calibration** — if a map entry is marked stale within 1 day of refresh across 5 consecutive traces, its TTL is halved automatically on the next refresh.
3. **Skill effectiveness metric** — if a skill's verification step consistently fails, the spec flags the skill for human review.
4. **Cross-agent consistency check** — if two agents ran the same task on the same repo state and loaded significantly different context, the diff is surfaced as a warning in the next refresh sweep.

**Trace schema v2 (V3 addition):**

```json
{
  "task_id": "add-feature",
  "skill_id": "add-endpoint",
  "run_id": "uuid",
  "agent_id": "copilot|claude|custom",
  "repo_sha": "sha256:...",
  "timestamp": "2026-05-01T10:00:00Z",
  "steps": [
    {
      "action": "load",
      "file": "agent-context/intent/overview.md",
      "priority": "high",
      "reason": "task declared",
      "stale": false
    }
  ],
  "files_loaded": ["..."],
  "files_skipped": ["..."],
  "stale_warnings": ["..."],
  "token_estimate": 4200,
  "verification_result": "pass|fail|partial",
  "feedback": {
    "context_useful": true,
    "skill_effective": true,
    "suggested_priority_adjustments": []
  }
}
```

---

## Claim-Level Provenance (V3)

V2 required provenance at the file level. V3 requires it at the **claim level** for `intent/` and `skills/` — the two human-owned layers.

**Problem V2 left open:** An agent reads `intent/architecture.md` and finds the statement "Orders use eventual consistency." That claim might be six months old. The source file it was derived from (`src/Orders/OrderService.cs`) may have changed. File-level provenance would only tell you whether the *whole file* is stale, not whether *that specific claim* is stale.

**V3 solution — inline cite blocks:**

In `intent/` markdown files, claim blocks carry an optional `<!-- _cite -->` annotation:

```markdown
## Consistency Model

Orders use eventual consistency for write operations.
<!-- _cite: src/Orders/OrderService.cs#L42-L58, fingerprint: sha256:abc123, checked: 2026-05-01 -->
```

In `skills/` markdown files, constraint and verification steps carry the same annotation:

```markdown
### Constraints

- All endpoints must declare their contract DTO in `src/Contracts/`.
  <!-- _cite: src/Contracts/, fingerprint: sha256:def456, checked: 2026-05-01 -->
```

**Tooling:** A `ales check-claims` command (CLI or CI step) scans cite blocks, recomputes fingerprints of referenced source regions, and flags any that have drifted. This is the lightweight implementation of full claim-level provenance.

---

## Multi-Agent Concurrency Model (V3)

V2 assumed a single-agent environment. V3 defines the minimum rules for safe concurrent access.

**Three concurrency problems:**

| Problem | V3 rule |
|---|---|
| Two agents mark the same map entry stale simultaneously | Last-write-wins on `stale: true`. Once stale, only a refresh can clear it. No agent may set `stale: false` directly. |
| Two agents refresh the same entry concurrently | Refresh is idempotent. If two refresh sweeps overlap, the result is two identical re-derives. The `_meta.generated_at` timestamp is the tiebreaker for the index. |
| Agent A proposes an intent update while Agent B is mid-task using the current intent | Intent updates land in `_pending/` (see Approval UX below). The current intent file is immutable during any active task execution. An active task reads a snapshot; `_pending/` entries are never loaded mid-task. |

**Lock-free by design:** ALES does not introduce file locks or distributed coordination. The above rules are achievable with git's atomic write semantics and a single-commit-per-mark-stale discipline.

---

## Approval UX for Intent Drafts (V3)

V1 and V2 left this as an open question. V3 specifies a minimal, git-native answer.

**Flow:**

```
Agent proposes intent update
  └→ writes to agent-context/intent/_pending/{filename}-{run_id}.draft.md
  └→ emits a summary in the trace: "intent update pending review"

Human reviews the draft
  └→ accepts: renames/merges into agent-context/intent/{filename}.md, deletes draft
  └→ rejects: deletes draft

CI checks
  └→ presence of any file in _pending/ triggers a warning (not a block) in the PR
  └→ ales check-claims runs on the merged intent file after acceptance
```

**Why git-native:** No external approval service, no webhook, no database. The draft file is the PR artifact. The review is a code review. This is the minimal mechanism that works in a single-author POC and scales to a multi-author team without changing the spec.

---

## Cross-Repo Skill and Task Import (V3)

V2 noted a "skill/task marketplace" as a product-stage goal. V3 defines the **minimum import interface** — enough to share skills and tasks between repos without a hosted service.

**Concept:** A skill or task may declare an `origin` in its header. If `origin` is present, the local file is a fork of a shared definition. The fork may extend, restrict, or override steps.

**Skill header example:**

```json
{
  "id": "add-endpoint",
  "origin": {
    "source": "github:ales-shared/dotnet-skills/add-endpoint.skill.md",
    "version": "1.2.0",
    "forked_at": "2026-04-01",
    "local_overrides": ["constraints", "verification"]
  },
  "applies_to_task": "add-feature",
  "steps": ["..."],
  "constraints": ["..."],
  "verification": ["..."],
  "ask_when": ["..."]
}
```

**Import resolution rules:**

1. Local file always takes precedence over origin — the fork is the authoritative version for this repo.
2. `ales check-updates` compares the local skill against the origin version and reports diffs (not auto-updates).
3. A repo may pin to a specific origin version. The check command warns if the origin has a newer version.

**V3 scope:** `ales check-updates` is a CLI command. No hosted registry is required. The origin URL is a git reference (GitHub path, tag, or commit SHA). Fetching is the responsibility of the tool that runs the check.

---

## Execution Loop V3

```
1. RESOLVE     → match user request to a task in the task registry
               → check for cross-repo import if task not found locally
2. PLAN        → load task; select applicable skill; check _pending/ for blocking drafts
               → read last N traces for this task type (adaptive priority selector)
               → estimate token budget with priority-adjusted load order
               → check for stale context (fingerprint + TTL + trace-informed score)
3. LOAD        → load context files in priority order (static + trace-informed)
4. EXECUTE     → run steps in declared order
   ├─ on missing context   → escalate priority; load next tier
   ├─ on stale context     → emit stale warning; continue or stop per severity
   ├─ on ambiguous flow    → STOP; emit ask
   ├─ on claim cite drift  → emit cite drift warning; continue or stop per severity
   └─ on budget exceeded   → emit partial result + reason
5. VERIFY      → check output against expected_output_schema in the task definition
6. EMIT        → return result + trace v2 (task_id, skill_id, agent_id, files loaded,
               token estimate, verification result, feedback block)
7. FEEDBACK    → write trace to traces/{task_id}/{run_id}.trace.json
               → update traces/_index.json
               → if TTL calibration triggered → write proposed _meta update to _pending/
```

**New guaranteed behaviors (V3 additions):**

- Adaptive priority selector is applied before budget estimation on every task execution.
- Claim cite drift warnings are emitted when `ales check-claims` data is available.
- Trace feedback block is mandatory; agents may leave `suggested_priority_adjustments` empty but not absent.
- `_pending/` is checked at PLAN time; a blocking draft (explicitly flagged `"blocking": true`) halts execution.

**Action vocabulary (V3 additions):**

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
| `write_trace` | *(V3)* Write completed trace to `traces/` layer |
| `check_claims` | *(V3)* Verify cite block fingerprints against current source |
| `propose_intent` | *(V3)* Write a draft to `intent/_pending/` |
| `import_skill` | *(V3)* Resolve a cross-repo skill reference |

---

## Folder Structure (V3)

```
/agent-context/
├── ales.manifest.json              ← spec version, repo SHA, source-of-truth policy,
│                                     task registry, import registry
│
├── intent/                         ← authored, slow-changing; claim-level _cite blocks
│   ├── overview.md
│   ├── architecture.md
│   ├── conventions.md
│   ├── glossary.md
│   ├── decisions/                  ← ADR-style "why we chose X"
│   └── _pending/                   ← agent-proposed draft updates (human approval required)
│
├── map/                            ← derived, agent-maintained; trace-informed TTL
│   ├── modules.json
│   ├── apis.json
│   ├── flows/
│   └── _provenance.json
│
├── skills/                         ← project-specific recipes; claim-level _cite blocks;
│   ├── *.skill.md                    optional origin pointer for cross-repo import
│   └── _pending/                   ← agent-proposed draft skills (human approval required)
│
├── tasks/                          ← agent execution procedures; optional origin pointer
│   └── *.task.json
│
└── traces/                         ← (V3 new) managed trace layer
    ├── _index.json
    └── {task_id}/
        └── {run_id}.trace.json
```

---

## Schema Extensions (V3)

| Schema | V3 additions |
|---|---|
| **Manifest** | + `import_registry[]` (cross-repo origin sources), + `trace_retention_days` |
| **Task** | + `adaptive_priority: true\|false`, + `origin` block |
| **Skill** | + `origin` block, + `_cite` in constraints and verification |
| **Map entry** | + `dependency_graph[]` (which other map entries this entry depends on, for cascade marking) |
| **Provenance `_meta`** | + `trace_informed_ttl_days` (calibrated; separate from `ttl_days` authored default) |
| **Trace** | Full v2 schema (see above); mandatory `feedback` block |
| **Claim cite** | `_cite: <source_path>#<line_range>, fingerprint: <sha>, checked: <date>` inline annotation |

---

## Evaluation (V3 — Run, Don't Just Design)

V2 designed the protocol. V3 runs it and reports real numbers.

**Benchmark setup:**

- Two real software repos (different tech stacks — e.g., .NET + TypeScript/Node — to test model-portability).
- Two agent runtimes per repo (e.g., GitHub Copilot + Claude Code).
- Four task types: `add-feature`, `debug-test-failure`, `refresh-map`, `add-skill`.
- Three conditions per task: No-ALES, Single-instruction-file, ALES-v3.

**Metrics to report (V3 commitments):**

| Metric | V3 target |
|---|---|
| Files loaded | Report mean ± std across 10 runs per condition |
| Token estimate | Report mean ± std; compute cost delta vs. full-repo-dump baseline |
| Task success rate | Pass/fail judged by build + test result and human eval |
| Trace completeness rate | % of runs with a well-formed, complete trace |
| Stale detection accuracy | % of stale warnings that correctly identified real drift |
| Cross-agent consistency | Jaccard similarity of files loaded between two agents on same task/state |
| Claim cite drift detection | % of drifted claims surfaced by `ales check-claims` vs. manual audit |

**Paper revision:** The revised paper includes a results section with these numbers, an honest discussion of where ALES fell short (e.g., claim-level provenance false-negative rate), and a research agenda for V4.

---

## Security Model (V3)

V2 named the minimum POC security awareness. V3 operationalizes it.

| Control | V3 requirement |
|---|---|
| `tasks/` and `skills/` are privileged surfaces | CODEOWNERS rule required in any multi-author repo; CI blocks merge without approval |
| Cross-repo imports are pinned | `origin.version` must be a pinned tag or commit SHA, not a floating branch |
| `_pending/` drafts are not loaded mid-task | Enforced by the PLAN step — spec declares `_pending/` as non-loadable during execution |
| Traces may contain sensitive paths | `ales.manifest.json` may declare `trace_redact_patterns[]` (glob list) to redact file paths before writing |
| No secrets or credentials in `/agent-context/` | Documented rule; `ales validate` checks for common secret patterns (API keys, connection strings) |

---

## Deferred To Later Stages

| Deferred feature | Target stage |
|---|---|
| Full RBAC and least-privilege controls | Enterprise adoption |
| Audit logging and compliance | Regulated environments |
| Signed or trusted context files | Multi-author adversarial environments |
| Tenant and project isolation | Hosted multi-org deployment |
| Observability dashboards | Production monitoring |
| Hosted skill/task registry service | Product stage |
| Azure or cloud deployment | Product stage |
| Non-code production validation (animation, stats) | Post-paper expansion |
| Distributed multi-agent orchestration beyond concurrency rules | V4 research target |
| Full probabilistic provenance (sentence → code region → author) | V4 research target |

---

## Open Questions For V3

1. **Adaptive priority selector implementation** — rule-based (frequency count from traces) or lightweight ML? What is the minimum that is both useful and auditable?
2. **Claim cite tooling** — should `ales check-claims` be a standalone CLI, a CI action, or a language server plugin? What is the minimum viable implementation?
3. **`_pending/` blocking semantics** — when should a draft be flagged `"blocking": true`? Human-set or automatically set when the draft affects a claim the current task depends on?
4. **Cross-repo import security** — how should a repo verify the integrity of an imported skill? Should `ales.manifest.json` pin a content hash of the remote file, not just a version tag?
5. **Trace retention and privacy** — traces contain file paths, potentially sensitive data. What is the right default `trace_retention_days` and redaction policy for open-source vs. private repos?
6. **Multi-runtime consistency** — if GitHub Copilot and Claude Code both run the same task and produce divergent traces, who arbitrates which priority calibration wins?
7. **Second POC repo selection** — what second tech stack best demonstrates model-portability without duplicating the first repo's domain?
8. **Sub-agent spawning protocol** — when a primary agent delegates a scoped sub-task, how does it pass the context snapshot boundary? Does the sub-agent receive the full loaded context or only the files declared for that step?
9. **Tool task isolation** — tool tasks are declared read-only in V3. Should a tool task be able to escalate to a write (e.g., `mark_stale`) with explicit primary agent delegation, or is the boundary absolute?

---

## Next Artifacts

In order:

1. `planV3.md` — this file.
2. `schema-roadmap-v3.md` — extend V2 schema roadmap with trace schema v2, claim cite format, origin block, and manifest import registry.
3. `evaluation-results.md` — run the V2 evaluation protocol; record actual numbers.
4. `claim-cite-spec.md` — precise syntax, tooling contract, and CI integration for claim-level provenance.
5. `multi-agent-rules.md` — detailed concurrency rules, conflict scenarios, and lock-free protocol.
6. `cross-repo-import-spec.md` — import interface, pinning rules, `ales check-updates` contract.
7. Update `paper-plan.md` — revise for empirical results section, V3 contributions, research agenda.
