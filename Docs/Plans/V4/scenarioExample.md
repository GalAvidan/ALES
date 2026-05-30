# ALES v4 — Scenario Examples

**Purpose:** Concrete walkthroughs of the Scope Stack model against an imaginary software system. Demonstrates what is stored where, how the agent navigates the layers, how context is assembled, and how conflicts are resolved.

---

## The Setup: Acme Commerce

**Company:** Acme Corp  
**Program:** Acme Commerce (the online store)  
**Repos:**

| Repo | Purpose |
|---|---|
| `acme-api` | Backend HTTP API (.NET) |
| `acme-web` | Storefront frontend (Next.js) |
| `acme-infra` | Bicep / IaC and pipelines |

**Projects inside `acme-api`:**

| Project | Purpose |
|---|---|
| `Catalog` | Product listing and search |
| `Orders` | Order lifecycle state machine |
| `Payments` | Payment capture, refund, audit |

**Scope stack:**

```
Ecosystem    (public shared ALES blocks)
     ↓
Enterprise   (acme-corp/platform-context)
     ↓
Workspace    (acme-commerce/program-context)
     ↓
Repo         (acme-api)
     ↓
Project      (Orders)
```

---

## Where Each Scope Lives and What It Contains

```
acme-corp/platform-context/
└── agent-context/                        ← ENTERPRISE scope
    ├── ales.manifest.json
    ├── intent/
    │   ├── overview.md                   "Acme Corp runs 4 programs. 300 engineers."
    │   ├── security-policy.md            AUTH-001: all endpoints require Entra ID auth.
    │   ├── platform-catalog.md           Approved: SQL Server, Cosmos, Entra ID, Azure PaaS.
    │   └── decisions/
    │       └── 2025-no-shared-db.md      ADR: each service owns its own DB schema.
    └── skills/
        └── cross-repo-feature-rollout.skill.md


acme-commerce/program-context/
└── agent-context/                        ← WORKSPACE scope
    ├── ales.manifest.json
    ├── intent/
    │   ├── program-overview.md           "3 repos: acme-api, acme-web, acme-infra."
    │   └── shared-contracts.md           Order DTO owned by acme-api; consumed by acme-web.
    ├── map/
    │   └── dependency-graph.json         acme-web depends on acme-api; both depend on acme-infra.
    ├── skills/
    │   └── add-feature-across-repos.skill.md
    └── tasks/
        └── coordinated-release.task.json


acme-api/
└── agent-context/                        ← REPO scope
    ├── ales.manifest.json                (see ales.manifest.json in this folder)
    ├── _index.json                       (see _index.json in this folder)
    ├── intent/
    │   ├── overview.md                   "acme-api: Clean Arch, .NET 9, MediatR."
    │   ├── architecture.md               Layers, CQRS, folder layout.
    │   └── conventions.md                Naming, DTO location, error model.
    ├── map/
    │   ├── modules.json                  Catalog, Orders, Payments modules.
    │   └── apis.json                     14 endpoints across 3 controllers.
    ├── skills/
    │   ├── add-endpoint.skill.md         .NET-specific endpoint recipe.
    │   ├── run-tests.skill.md            dotnet test, LocalDB, 80% threshold.
    │   └── deploy.skill.md               Azure Pipelines, staging → prod gate.
    └── traces/
        └── add-endpoint/...


acme-api/src/Orders/
└── agent-context/                        ← PROJECT scope (Orders)
    ├── ales.manifest.json
    ├── intent/
    │   ├── domain.md                     Order states: Draft → Placed → Paid → Shipped → Cancelled.
    │   ├── invariants.md                 Cancelled orders cannot be re-paid.
    │   └── decisions/
    │       └── 2026-cancellation-flow.md ADR: cancel allowed from Placed only.
    ├── skills/
    │   └── add-order-state-transition.skill.md
    └── traces/
        └── ...
```

---

## Scenario 1 — Single-Project Task

**Request:** `"Add a Cancel Order endpoint to acme-api"`  
**Scopes involved:** Project (Orders), Repo (acme-api), Enterprise  
**No workspace/cross-repo work needed.**

### Step 1 — DISCOVER

Agent is working inside `acme-api/src/Orders/`. It walks up to find:

```
src/Orders/agent-context/ales.manifest.json   → project:Orders
        parent_scope →
acme-api/agent-context/ales.manifest.json     → repo:acme-api
        parent_scope →
acme-commerce/.../ales.manifest.json          → workspace:acme-commerce
        parent_scope →
acme-corp/.../ales.manifest.json              → enterprise:acme-corp
```

### Step 2 — INDEX (cheap)

Agent loads `_index.json` for each scope. Headlines only.

| Scope | Index size |
|---|---|
| enterprise | ~400 tokens |
| workspace | ~350 tokens |
| repo | ~650 tokens (see `_index.json`) |
| project:Orders | ~300 tokens |
| **Total** | **~1,700 tokens** |

Agent now knows every block that exists without having loaded any of them.

### Step 3 — RESOLVE

Agent matches "add an endpoint" → `task:add-feature` in the repo registry.

Skill selector: task says `invoke_skill: add-endpoint`. Repo index shows `skill:add-endpoint` exists. Project index shows `skill:add-order-state-transition` which `specializes` it.

### Step 4 — PLAN: token budget computed

| Block | Loaded as | Tokens |
|---|---|---|
| `enterprise/intent/security-policy.md` | Summary + AUTH-001 excerpt | 600 |
| `workspace/intent/shared-contracts.md` | Cited section only | 300 |
| `repo/intent/architecture.md` | Summary | 500 |
| `repo/intent/conventions.md` | Full | 950 |
| `repo/skills/add-endpoint.skill.md` | Full | 820 |
| `project/intent/domain.md` | Full | 800 |
| `project/intent/invariants.md` | Full | 500 |
| `project/intent/decisions/2026-cancellation-flow.md` | Full | 420 |
| `project/skills/add-order-state-transition.skill.md` | Full | 640 |
| Last 3 repo traces (add-endpoint) | Summary | 900 |
| **Context sub-total** | | **~6,430** |
| Source code files (controller, handlers, DTOs, tests) | Loaded during execute | ~18,000 |
| Conversation | | ~3,000 |
| **Total** | | **~27,430** |

Well within budget. ~93k headroom for iterative work.

### Step 5 — EXECUTE

Agent follows `add-order-state-transition.skill` (which `specializes` `add-endpoint.skill`):

| Decision | Sourced from |
|---|---|
| Endpoint path: `POST /orders/{id}/cancel` | Repo conventions (REST resource pattern) |
| Auth required (Entra ID) | Enterprise security-policy — AUTH-001 |
| Transition only from `Placed` state | Project invariant + 2026 ADR |
| Must emit `OrderCancelled` domain event | Project skill constraint |
| DTO goes in `src/Contracts/` | Repo conventions |
| Unit test + integration test required | Repo run-tests skill |
| Test command: `dotnet test` | Repo run-tests skill |

No cross-repo work. No workspace blocks needed beyond the shared-contracts reference check.

### Step 6 — VERIFY + EMIT

```json
{
  "task_id": "add-feature",
  "skill_id": "add-order-state-transition",
  "scopes_loaded": ["enterprise", "repo:acme-api", "project:Orders"],
  "files_loaded": [
    "enterprise/intent/security-policy.md (excerpt)",
    "repo/intent/conventions.md",
    "repo/skills/add-endpoint.skill.md",
    "project/intent/domain.md",
    "project/intent/invariants.md",
    "project/skills/add-order-state-transition.skill.md"
  ],
  "token_estimate": 27430,
  "verification_result": "pass"
}
```

Trace written to:
- `acme-api/agent-context/traces/add-feature/{run_id}.trace.json` (repo scope)
- `acme-api/src/Orders/agent-context/traces/add-feature/{run_id}.trace.json` (project scope)

---

## Scenario 2 — Cross-Repo Task

**Request:** `"Add a Cancel Order button in the web storefront. The API endpoint already exists."`  
**Scopes involved:** Project (Orders in acme-api), Repo (acme-api), Repo (acme-web), Workspace, Enterprise  
**This requires workspace-level coordination.**

### Step 1 — DISCOVER

Same upward walk as Scenario 1, but now the task spans two repos. The `workspace/tasks/coordinated-release.task.json` is found in the workspace scope.

### Step 2 — INDEX

Workspace scope is now active. Agent also loads `_index.json` for `acme-web`.

| Scope | Index tokens |
|---|---|
| enterprise | ~400 |
| workspace | ~420 |
| repo:acme-api | ~650 |
| project:Orders | ~300 |
| repo:acme-web | ~580 |
| project:storefront (acme-web) | ~280 |
| **Total** | **~2,630** |

### Step 3 — RESOLVE

"Add a Cancel button" + "API already exists" → workspace task `coordinated-release` is invoked.

Inside `coordinated-release`, step 1 is: "verify API contract exists and is not stale."

### Step 4 — PLAN

Workspace block `shared-contracts.md` becomes a first-class load because it owns the Order DTO definition that both repos share:

| Block | Scope | Loaded as | Tokens |
|---|---|---|---|
| `enterprise/security-policy.md` | Enterprise | Excerpt | 600 |
| `workspace/shared-contracts.md` | Workspace | **Full** | 1,200 |
| `workspace/dependency-graph.json` | Workspace | Full | 400 |
| `repo:acme-api/map/apis.json` | Repo | Summary (verify cancel endpoint exists) | 400 |
| `repo:acme-web/intent/architecture.md` | Repo | Summary | 500 |
| `repo:acme-web/conventions.md` | Repo | Full | 780 |
| `repo:acme-web/skills/add-action-button.skill.md` | Repo | Full | 710 |
| `project:storefront/intent/domain.md` | Project | Full | 650 |
| Recent traces (both repos) | Memory | Summary | 1,200 |
| **Context sub-total** | | | **~6,440** |
| Source files (acme-web components, types) | Execute-time | | ~20,000 |
| **Total** | | | **~29,440** |

### Step 5 — EXECUTE

**Leg 1 — Verify API side (acme-api, read-only)**

Agent loads `map/apis.json` from `repo:acme-api`. Confirms `POST /orders/{id}/cancel` exists and is not stale. No changes needed. Emits verification record.

**Leg 2 — Implement web side (acme-web)**

Agent follows `add-action-button.skill` from `repo:acme-web`:

| Decision | Sourced from |
|---|---|
| Cancel DTO type comes from `acme-api/src/Contracts/CancelOrderResponse.ts` | Workspace shared-contracts |
| Button visible only if order state is `Placed` | Workspace shared-contracts (state enum) |
| API call uses the shared API client utility | acme-web repo conventions |
| Must add a unit test for the component | acme-web run-tests skill |

### Step 6 — Conflict Example

What if `acme-web/src/storefront/agent-context/intent/domain.md` says:

> "Cancel button should be shown for all non-delivered orders."

But `workspace/shared-contracts.md` says:

> "Cancel is only valid from `Placed` state. Consumer UIs must reflect this."

**Conflict detected.** Precedence rule: `workspace wins over project for shared contracts`.

Agent does not silently pick a side. It emits:

```json
{
  "conflict": {
    "id": "CONFLICT-001",
    "category": "shared-contract-vs-project-assumption",
    "winner": "workspace:acme-commerce/shared-contracts",
    "loser": "project:storefront/intent/domain.md",
    "reason": "Workspace wins for shared contract definitions.",
    "action": "continue — apply workspace rule",
    "recommendation": "Update project:storefront/intent/domain.md to reflect the correct state constraint."
  }
}
```

The agent continues using the workspace rule, writes the conflict to the trace, and adds a `_pending/` update proposal for the stale project intent file.

### Step 7 — EMIT + traces

```json
{
  "task_id": "coordinated-release",
  "scopes_loaded": ["enterprise", "workspace:acme-commerce", "repo:acme-api", "project:Orders", "repo:acme-web", "project:storefront"],
  "conflicts_resolved": ["CONFLICT-001"],
  "token_estimate": 29440,
  "verification_result": "pass"
}
```

Trace written to:
- `acme-web/agent-context/traces/coordinated-release/{run_id}.trace.json`
- `acme-commerce/program-context/agent-context/traces/coordinated-release/{run_id}.trace.json` (workspace scope, because this was multi-repo)

---

## Scenario 3 — Budget Pressure

**Request:** `"Refactor the entire Orders service to use Minimal APIs instead of controllers"`  
**Issue:** This is a large change. The full Orders project context + repo conventions + tests + source files would exceed budget if loaded naively.

### What happens at PLAN time

Agent estimates budget:

```
project:Orders/intent/domain.md           → 800 tokens
project:Orders/intent/invariants.md       → 500 tokens
project:Orders/intent/decisions/ (all)    → 1,400 tokens
repo/intent/architecture.md              → 1,800 tokens (full — needed, architecture change)
repo/intent/conventions.md              → 950 tokens
repo/skills/add-endpoint.skill.md        → 820 tokens
repo/map/modules.json                    → 680 tokens
repo/map/apis.json                       → 1,200 tokens
enterprise/security-policy.md           → 600 tokens (excerpt)
Recent traces × 5                        → 1,500 tokens
─────────────────────────────────────────────────────
Context sub-total                        → ~10,250
Source code (entire Orders module)       → ~60,000 (estimated)
Tests                                    → ~20,000 (estimated)
─────────────────────────────────────────────────────
Estimated total                          → ~90,250
```

Within 200k limit, but the agent forecasts it will need multiple sub-tasks (load → refactor one handler → verify → next handler).

**Budget rule applied:** Agent does not load all source files upfront. It:

1. Loads context blocks fully (10,250 tokens).
2. Loads source files **per handler**, one at a time during execute.
3. Uses the trace from each handler's refactor to inform the next.
4. Emits a partial trace after each handler with a `continue: true` flag.
5. Final trace aggregates all partials.

This is the **iterative loading** pattern. No hallucinations from truncation. No silent budget overflow. Bounded by design.

---

## Scenario 4 — Rule Conflict That Requires a Human

**Request:** `"Add a payment method endpoint that doesn't require auth — it's a public webhook from Stripe"`

Agent loads:
- `enterprise/security-policy.md` — AUTH-001 (blocking): all endpoints require Entra ID auth.
- `project:Payments/intent/domain.md` — describes Stripe webhook inbound flow.
- `repo/skills/add-endpoint.skill.md` — default: auth required.

**Conflict:**

The user's request contradicts AUTH-001 (enterprise, `severity: blocking`).

The agent cannot override an enterprise `blocking` rule. It does not guess. It does not silently add auth where it would break Stripe's webhook flow.

It emits:

```
CONFLICT: AUTH-001 (blocking) prevents creating an unauthenticated endpoint.
Enterprise policy: all endpoints require Entra ID authentication.
User request explicitly requires no auth (Stripe webhook).

Options:
  A. Add auth using Stripe's webhook signature verification instead of Entra ID.
     (Enterprise policy may accept this as equivalent for inbound webhooks — check security-policy.md#L78)
  B. File an exception ADR. Enterprise security review required before proceeding.
     (Add to agent-context/intent/_pending/ with severity:blocking flag)
  C. Abandon the unauthenticated approach.

Action required: This step is halted. Please choose A, B, or C.
```

The agent stops, presents options sourced from the actual policy document, and waits for a human decision.

**This is the correct behavior.** It is also auditable: the `ask` event is recorded in the trace with the exact conflict, the rule that blocked it, and the options offered.

---

## Key Observations Across All Scenarios

| Pattern | Observation |
|---|---|
| **Index first** | Agent never loads a full file to discover what exists. _index.json headlines are sufficient for routing. |
| **Scope narrows what's relevant** | Single-project task never needed workspace blocks. Cross-repo task activated workspace coordination. Only the relevant scopes participate. |
| **Connections, not searches** | Every load decision followed a typed connection (`governed_by`, `specializes`, `depends_on`). The agent never searched the filesystem. |
| **Conflicts are explicit** | Conflicts were detected, categorized, ruled on, recorded. No silent overrides. |
| **Budget is predictable** | The tiered loading model kept context well under 30k for single-repo tasks and under 30k for cross-repo tasks — not because content was cut, but because summaries are sufficient for most scope tiers. |
| **Traces close the loop** | Every scenario produced a trace that records scope, blocks loaded, conflicts, and verification. Future runs for the same task type on the same scope start with better-informed priority selection. |
