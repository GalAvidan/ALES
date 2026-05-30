# ALES v4 — Design Plan

**Status:** Proposal  
**Stage:** Scope Stack — Composition, Precedence, and Cross-Scope Connectivity  
**Supersedes:** `../V3/planV3.md`  
**Inputs:** `../V3/planV3.md`, `../V2/planV2.md`, `../V1/planV1.md`, `../paper-plan.md`

---

## Why V4 Exists

V3 made ALES empirically grounded. It added:
- `traces/` as a first-class memory layer.
- Claim-level provenance (`_cite` blocks).
- Multi-agent concurrency rules.
- Cross-repo import with `origin` blocks.
- An executor taxonomy and `_pending/` approval flow.

What V3 still assumed was a single-scope world: one `/agent-context/` folder, one repo, one set of stakeholders. The folder contents were the layers.

V4 challenges that assumption.

> The folders (`intent/`, `map/`, `skills/`, `tasks/`, `traces/`) are **content types**, not the layering itself.
> The real layering is **organizational scope**: project, repo, workspace/program, enterprise, ecosystem.
> Any scope can contain any content type.
> An agent execution assembles a context stack by composing blocks from all relevant scopes.

This is the Scope Stack model.

---

## What Changed V3 → V4

| V3 | V4 |
|---|---|
| One `/agent-context/` folder per repo | `agent-context/` can live at any scope level |
| Four/five content folders are the layers | Content folders are block types; scope is the axis |
| Cross-repo import is a V3 add-on | Cross-scope composition is a first-class design |
| Multi-agent concurrency rules | Scope-level authority and precedence rules |
| Security operationalized per repo | Enforcement defined across all four layers per scope |
| Trace memory feeds one repo's adaptive selector | Traces scoped per execution; aggregatable upward |
| No concept of enterprise or program context | Enterprise, workspace, project scopes fully defined |

---

## Core Thesis (Refined from V3)

> ALES is a composable, scope-aware context stack. Each organizational scope — project, repo, workspace, enterprise, or shared ecosystem — can provide knowledge, rules, tasks, and memory blocks. During execution, an agent assembles the relevant blocks into a task-specific context stack. Broader scopes provide governance, reusable defaults, and coordination. Narrower scopes provide local truth, implementation detail, and execution memory. Typed connections describe how scopes relate. Precedence rules describe what happens when they conflict.

Four preserved properties, one new:

| Property | V4 definition |
|---|---|
| **Bounded context** | Agent loads indexed summaries by default; full blocks only for actively needed content. Budget is declared per scope tier. |
| **Auditable traversal** | Trace records scope, block, load reason, and priority for every file loaded. Scoped traces are diffable. |
| **Model-portable** | Same scope stack is readable by any conformant agent runtime. |
| **Feedback-aware** | Traces feed back into scope-level priority calibration and enterprise-level metrics. |
| **Scope-composable** (new) | A context stack is assembled from multiple scope layers. Connections are typed. Conflicts are resolved by precedence rules, not silently. |

---

## The Two-Axis Model

ALES has two orthogonal axes:

**Axis 1 — Content type** (what kind of block this is):

| Content type | Folders | Meaning |
|---|---|---|
| **Knowledge** | `intent/` + `map/` | Why things exist, what exists, where things live |
| **Rule** | `skills/` | How work must be done in this scope |
| **Task** | `tasks/` | Reusable generic or scope-specific procedures |
| **Memory** | `traces/` | What was done before, what worked, traces |

**Axis 2 — Scope layer** (where the block applies):

| Scope | Authority | Typical content |
|---|---|---|
| **Ecosystem** | Lowest (default/template) | Generic reusable tasks, skill templates |
| **Enterprise** | Highest for policy | Security, compliance, approved platforms, ownership |
| **Workspace / Program** | Coordination authority | Multi-repo coordination, shared contracts, release order |
| **Repo** | Operational authority | Build, test, deploy, repo conventions, repo-local skills |
| **Project** | Implementation authority | Domain model, invariants, local patterns |

Every cell in the two-axis grid is valid:

|  | Knowledge | Rule | Task | Memory |
|---|---|---|---|---|
| Ecosystem | Generic patterns | Recommended conventions | Generic procedures | — |
| Enterprise | Platform catalog, security policy | Compliance constraints | Cross-program rollout | Org-level metrics |
| Workspace | Repo map, shared contracts | Release sequencing rules | Coordinated release | Program traces |
| Repo | Architecture, folder layout | Build/test/deploy conventions | Repo-specific task overrides | Repo traces |
| Project | Domain model, invariants | Local workflow patterns | Project-specific procedures | Project traces |

---

## Scope Definitions

### Ecosystem

**Lives at:** A public or shared-internal registry (e.g., `github:ales-shared/...`).  
**Contains:** Generic reusable tasks and skill templates that make no assumptions about a specific organization or tech stack.  
**Does not contain:** Company secrets, repo-specific commands, org-specific conventions.  
**Authority:** Lowest. Any local scope may override, fork, or pin.  
**V3 connection:** This is the V3 cross-repo import source, now given a formal scope name.

### Enterprise

**Lives at:** A central organization repo (e.g., `acme-corp/platform-context/agent-context/`).  
**Contains:**
- `intent/` — company-wide architecture, approved platforms, system ownership model.
- `skills/` — cross-repo feature rollout, enterprise logging standard, audit trail requirements.
- `tasks/` — enterprise change governance procedures.
- No `traces/` — enterprise reads aggregated metrics derived from lower-scope traces.

**Does not contain:** Detailed implementation knowledge of any one project, repo-local commands.  
**Authority:** Highest for security, compliance, and platform constraints. Cannot be overridden at lower scopes without an explicit exception ADR.

### Workspace / Program

**Lives at:** A coordination repo for the program (e.g., `acme-commerce/program-context/agent-context/`).  
**Contains:**
- `intent/` — program overview, which repos participate, contract ownership map.
- `map/` — dependency graph across repos.
- `skills/` — multi-repo change coordination procedures.
- `tasks/` — coordinated release, cross-repo feature rollout.
- `traces/` — traces for multi-repo executions.

**Does not contain:** Global company security policy, repo-specific build commands, individual project domain rules.  
**Authority:** Coordination authority. Wins on release order, shared contracts, and cross-repo dependency rules.

### Repo

**Lives at:** `/agent-context/` at the root of the repository (current V1/V2/V3 convention, preserved).  
**Contains:**
- `intent/` — repo purpose, architecture, tech stack, conventions, ADRs.
- `map/` — module listing, API surface, data flows.
- `skills/` — repo-local recipes (how to add an endpoint, run tests, deploy here).
- `tasks/` — repo-local task overrides of ecosystem tasks.
- `traces/` — repo-level execution traces.

**Does not contain:** Enterprise policy definitions, workspace release schedules, other repos' conventions.  
**Authority:** Operational authority for build, test, deploy, and code structure.

### Project

**Lives at:** `/src/{ProjectName}/agent-context/` inside the repo.  
**Contains:**
- `intent/` — feature/domain purpose, domain model, business rules, invariants, local ADRs.
- `skills/` — project-specific workflow recipes.
- `traces/` — project-scoped task traces.

**Does not contain:** Repo-wide deployment, other projects' domain rules, enterprise policy.  
**Authority:** Implementation authority. Wins on local design decisions and domain conventions.

---

## Typed Connections

Connections between blocks are explicit and typed. They are declared in each block's header and in the manifest. They are what let the agent walk the stack as a graph, not a folder crawl.

| Connection type | Meaning | Example |
|---|---|---|
| `governed_by` | Lower scope must obey higher scope's rule | Repo endpoint skill governed by enterprise security policy |
| `depends_on` | One block/scope needs another to be correct | Web project depends on API contract repo |
| `provides` | A scope exposes something others consume | Auth repo provides identity service |
| `consumes` | A project/repo uses another scope's capability | Frontend consumes billing API |
| `overrides` | Narrower scope specializes a broader default | Repo overrides shared `add-endpoint` skill |
| `imports` | Local block forks/pins a shared block | Repo imports ecosystem task template |
| `specializes` | Narrower block extends a broader block for local needs | Project skill specializes repo skill |
| `implements` | Local code fulfills a declared contract | Repo implements enterprise logging standard |
| `observed_by` | Traces from this scope feed a higher-scope metric | Project traces observed by enterprise metrics |

---

## Precedence Rules

Precedence is **category-specific**, not a simple ranking:

| Conflict category | Winner | Rationale |
|---|---|---|
| Factual system behavior | Code / tests / runtime | Always; ground truth |
| Security / compliance | Enterprise | Cannot be overridden without explicit exception ADR |
| Cross-repo coordination | Workspace | Coordination authority |
| Build / test / deploy | Repo | Operational authority |
| Local design and implementation | Project | Implementation authority |
| Imported shared templates | Local repo/project | Local fork always takes precedence |
| Conflict outside known categories | Agent must stop and ask | Do not guess |

The agent never silently resolves a conflict. If no precedence rule covers the conflict, it emits a conflict warning and halts the step with `ask`.

---

## Context Assembly

Agent execution assembles the stack **broad to specific**, but executes **specific to broad**:

```
1. DISCOVER   → locate nearest agent-context/ in working dir
             → walk parent_scope chain upward to find all ancestor scopes
             → walk child_scopes downward to find project scope(s) for affected area

2. INDEX      → load ales.manifest.json and _index.json for each in-scope layer  (~2-5k tokens total)

3. RESOLVE    → match user request to a task in the registry
             → identify affected project(s), repo(s), workspace if cross-repo

4. PLAN       → load task + applicable skill (full)
             → read last N traces for this task type across affected scopes
             → compute scope budget from manifest context_budget
             → load blocking constraints from all scopes
             → check _pending/ at all scopes for blocking drafts

5. LOAD       → load context blocks in priority order:
             │  enterprise policy and constraints (summary)
             │  workspace coordination rules (summary)
             │  repo architecture + conventions (summary, expand on demand)
             │  project domain + invariants (full)
             └  relevant traces (summary)

6. EXECUTE    → run task steps; invoke skills; apply typed connections
   ├─ on missing context       → escalate priority; load next tier
   ├─ on stale context         → emit stale warning; continue or stop per severity
   ├─ on cite drift            → emit cite drift warning
   ├─ on precedence conflict   → emit conflict + category; if no rule applies → ask
   └─ on budget exceeded       → emit partial + reason; do not silently truncate

7. VERIFY     → check output against expected_output_schema in task definition

8. EMIT       → return result + trace v3

9. FEEDBACK   → write trace to traces/ at the scope where work occurred
             → if multi-repo: write summary trace at workspace scope
             → update _index.json trace references
             → if TTL calibration triggered → write to _pending/ at affected scope
```

---

## Token Budget Model

Context is scarce. The stack must not blow the budget. Three mechanisms:

**1. Tiered block representation**

| Tier | Purpose | Typical size |
|---|---|---|
| Headline | One-line summary in `_index.json` | ~20 tokens |
| Summary | Block essentials, agent-generated | 200–500 tokens |
| Full | Complete file content | varies |

Indices are always loaded as headlines. Blocks are loaded as summaries unless the task explicitly requires full content.

**2. Per-scope budget allocation (declared in manifest)**

```json
"context_budget": {
  "total_tokens": 120000,
  "per_scope": {
    "enterprise":  { "max": 4000,  "default_tier": "summary" },
    "workspace":   { "max": 6000,  "default_tier": "summary" },
    "repo":        { "max": 25000, "default_tier": "summary" },
    "project":     { "max": 40000, "default_tier": "full" },
    "memory":      { "max": 8000,  "default_tier": "summary" }
  },
  "reserve_for_response": 30000
}
```

Enterprise gets the smallest allocation because its content is policy constraints that compress well. Project gets the largest because it is the active working surface.

**3. Hard rules against speculative loading**

- Never load a full enterprise file; load only cited rule excerpts unless a blocking constraint requires full context.
- Never load more than one full scope tier without an explicit escalation event.
- If budget would be exceeded at PLAN time, drop to summary tier and emit a budget warning in the trace.
- The `ask` action is always preferred over speculative loading.

**Typical budget for a single-repo task:**

| Block set | Tokens |
|---|---|
| All scope indices (headlines) | ~3,000 |
| Active task + active skill (full) | ~2,500 |
| Blocking constraints (cited excerpts) | ~1,500 |
| Project domain + invariants (full) | ~6,000 |
| Repo architecture summary | ~3,000 |
| Workspace contracts (relevant section) | ~1,500 |
| Enterprise policy (cited rules only) | ~800 |
| Recent traces (last 3, summary) | ~2,000 |
| Source code under modification | ~20,000 |
| Conversation history | ~5,000 |
| **Total** | **~45,300** |

That leaves ~125k of headroom in a 200k window for iterative work without compaction.

---

## Navigation Model

The agent never browses the filesystem. It routes through a declarative graph.

**Step 1 — Entry point:** The manifest is always the entry point. The agent locates the nearest `ales.manifest.json` by walking up from the current working directory.

**Step 2 — Scope graph:** The manifest declares:
- `parent_scope` — git URL or relative path to parent scope's manifest.
- `child_scopes[]` — list of project subdirectory manifests within this scope.

The agent walks this graph to discover all relevant scopes without filesystem traversal.

**Step 3 — Block index:** Each scope's `_index.json` lists every block with headline, token count, tags, and content type. The agent reads this instead of listing the directory.

**Step 4 — Connection graph:** Block headers declare typed connections. The agent follows these edges to decide which additional blocks to load. No searching.

**Result:** Navigation is $O(\text{scopes} \times \text{relevant blocks})$, not $O(\text{all files in repo})$.

---

## Enforcement Model

Rules are enforced at four layers (defense in depth):

| Layer | Mechanism | Strength | What it catches |
|---|---|---|---|
| **Prompt** | Rule injected into context as blocking constraint | Weak — steering only | ~70% of violations on a strong model |
| **Validator** | `ales validate`, `ales check-claims`, `ales lint --rule {id}` run pre-execution | Strong — deterministic | Schema violations, secret patterns, cite drift, forbidden paths |
| **Gate** | Pre-commit hooks, CI checks, CODEOWNERS on `tasks/`, `skills/`, enterprise blocks | Strong — prevents merge | Post-generation policy violations |
| **Sandbox** | Tool-restricted runtime; file-write allowlist from manifest `writable_paths` | Strongest — prevents capability | Requires runtime cooperation; not always available |

Rule blocks declare their enforcement posture:

```yaml
id: AUTH-001
scope: enterprise
severity: blocking        # info | recommended | required | blocking
applies_to_actions: [add-endpoint, modify-handler]
text: "Every endpoint must require authentication via Entra ID."
source: enterprise:security-policy#L42
verification:
  - type: ast_check
    command: "ales lint --rule AUTH-001"
enforcement:
  prompt: blocking
  validator: ales-lint
  ci_gate: required
  codeowner: "@security-team"
```

One rule, four enforcement points, all referencing the same block.

---

## Folder Structure (V4)

The V3 folder structure is preserved at each scope. V4 adds:
- Multi-scope placement (the same structure can appear at project, repo, workspace, enterprise).
- `ales.manifest.json` extended with scope identity, parent/child pointers, import registry, and budget.
- `_index.json` per scope — generated, kept fresh by the `refresh-map` task.

```
{scope-root}/
└── agent-context/
    ├── ales.manifest.json          ← scope identity, parent/child, budget, registry, import_registry
    ├── _index.json                 ← generated; all blocks with headline + token count + tags
    ├── intent/
    │   ├── *.md                    ← with _cite blocks (V3)
    │   └── _pending/               ← draft updates (V3)
    ├── map/
    │   ├── *.json                  ← with _meta provenance (V3)
    │   └── _provenance.json
    ├── skills/
    │   ├── *.skill.md              ← with origin block + _cite
    │   └── _pending/
    ├── tasks/
    │   └── *.task.json             ← with origin block + adaptive_priority
    └── traces/
        ├── _index.json
        └── {task_id}/{run_id}.trace.json
```

---

## Action Vocabulary (V4 Additions)

All V3 actions are preserved. V4 adds:

| Action | Meaning |
|---|---|
| `discover_scope` | Walk parent/child manifest chain to identify all relevant scopes |
| `load_index` | Load `_index.json` headlines for a scope without loading full blocks |
| `resolve_conflict` | Apply precedence rules to a detected conflict; emit `ask` if no rule covers it |
| `assert_compliance` | Check that a planned action satisfies all blocking constraints in scope |
| `aggregate_traces` | Collect and summarize traces from child scopes (workspace or enterprise use) |

---

## Schema Extensions (V4)

| Schema | V4 additions |
|---|---|
| **Manifest** | + `scope_type` (project/repo/workspace/enterprise/ecosystem), + `parent_scope`, + `child_scopes[]`, + `context_budget` block, + `writable_paths[]` |
| **Block header** | + `scope` identifier, + `governed_by[]`, + `depends_on[]`, + `provides[]` |
| **Skill / Task** | + `specializes` (V4 sibling to `overrides`), + `severity` on individual constraints |
| **Rule block** | New: `id`, `scope`, `severity`, `applies_to_actions[]`, `enforcement{}` |
| **_index.json** | + `scope`, + per-block `headline`, `tokens`, `tags`, `content_type`, `connections_summary` |
| **Trace v3** | + `scopes_loaded[]` (list of scope IDs contributing to this execution) |

---

## Version Progression Summary

| Version | Main contribution |
|---|---|
| V1 | Four-layer content taxonomy (intent, map, skills, tasks) |
| V2 | Defensible POC contract; corrected claims; evaluation protocol |
| V3 | Memory layer (traces); claim provenance; multi-agent; cross-repo import |
| V4 | Scope stack (project/repo/workspace/enterprise/ecosystem); typed connections; precedence rules; navigation model; enforcement depth; tiered token budget |

---

## Open Questions for V4

1. **Scope discovery UX** — Should `ales init` scaffold the parent_scope chain automatically, or is it always a manual declaration?
2. **Enterprise manifest hosting** — Should the enterprise manifest be resolvable from a git URL, a well-known path convention, or a registry endpoint? What is the minimum that works without a hosted service?
3. **Aggregate trace protocol** — How does an enterprise-level context aggregate traces from hundreds of repos without privacy violations? Is opt-in redaction enough?
4. **Conflict resolution logging** — When the agent emits a conflict and asks, where does that conflict record live? In the trace? In `_pending/`? Both?
5. **Project scope granularity** — Is one `/agent-context/` per top-level project folder sufficient, or do very large projects need sub-project scopes? Where is the line?
6. **Ecosystem registry trust** — What is the minimum integrity model for an imported ecosystem block? Content-hash pinning is specified; is that sufficient for a research-stage ALES?

---

## Deferred to Later Stages

| Deferred feature | Target stage |
|---|---|
| Full RBAC and least-privilege controls | Enterprise adoption |
| Signed / trusted context files | Multi-author adversarial environments |
| Hosted enterprise manifest registry | Product stage |
| Hosted ecosystem skill/task registry with vetting | Product stage |
| Aggregate trace analytics dashboard | Production monitoring |
| Probabilistic provenance (sentence → author) | V5 research target |
| Distributed multi-agent orchestration beyond concurrency rules | V5 research target |
| Non-code domain production validation (animation, stats) | Post-paper expansion |
