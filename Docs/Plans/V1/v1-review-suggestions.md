# ALES V1 — Review Suggestions

## Purpose

This document converts the production-readiness critique of ALES V1 into an implementation roadmap.

The goal is not to weaken the ALES idea. The goal is to make the claims precise enough to survive real production use, enterprise review, and technical due diligence.

## Positioning Correction

ALES should not be positioned as a replacement for agent companies, IDEs, model providers, RAG systems, orchestration frameworks, or tool protocols.

ALES is a repository-side structured knowledge and execution contract that helps any capable agent work with better local context.

It should complement systems such as:

- GitHub Copilot
- Claude Code
- Cursor
- OpenAI agent runtimes
- Semantic Kernel
- LangGraph
- LlamaIndex
- MCP-based tool ecosystems
- enterprise RAG and search platforms

The stronger positioning is:

> ALES standardizes the repo-side knowledge layer that agents consume. Agent runtimes remain free to choose their own tools, APIs, models, planners, and execution mechanisms.

This matters because production users will not adopt ALES if it appears to compete with their existing agent stack. They may adopt it if it makes that stack more reliable, auditable, and cheaper to operate.

## Skill And Task Semantics

The V1 distinction between tasks and skills is useful, but it needs sharper wording.

A task defines a generic goal-oriented procedure. A skill defines how that procedure is performed in this project.

Different agents may use different APIs or tools to achieve the same goal. That does not invalidate the skill. The skill should describe the stable human workflow:

1. What context to load.
2. What steps to perform.
3. What constraints must be respected.
4. What evidence must be checked.
5. How to verify the result.
6. When to stop and ask.

The runtime adapter decides how those steps map to concrete tools.

For example, ALES should specify:

```text
To add an endpoint in this repo, update the controller, application service, contract DTO, tests, and OpenAPI surface. Run the relevant test command and verify the generated API contract.
```

ALES should usually not specify:

```text
Call vendor-specific tool X with method Y and parameter Z.
```

That lower-level behavior belongs in the agent runtime, adapter, MCP server, IDE integration, or CLI implementation.

## Claim Boundaries For V1

Some current V1 claims are directionally right but too broad for production review.

| Claim | V1-safe wording |
|---|---|
| Deterministic traversal | ALES can make context selection and file-load order auditable and reproducible when the loader is deterministic. |
| Reproducible outputs | ALES can improve repeatability, but output reproducibility also depends on model, prompt, temperature, tools, runtime state, and external services. |
| Model-agnostic | ALES is model-portable as a repo contract, but different agents will vary in tool use, instruction following, context handling, and reliability. |
| Provenance everywhere | V1 should require provenance for generated map entries and derived claims. Full claim-level provenance is a harder target and should be phased in. |
| Domain-general | The taxonomy may generalize beyond code, but V1 production validation should focus on software repositories first. |

The revised message should be strict:

> ALES does not make agents deterministic by itself. It makes the context contract explicit enough that deterministic loaders, validators, traces, and evaluations can be built around it.

## Industry Standards: Required Vs Later

V1 does not need every enterprise feature on day one. It does need the minimum controls that prevent the contract from becoming stale, unsafe, or unverifiable.

### Required For Credible V1

| Requirement | Why it is required | Complexity introduced | New risk |
|---|---|---|---|
| JSON schemas | Makes manifest, tasks, skills, maps, flows, provenance, and traces machine-checkable. | Schema design and migration discipline. | Bad schemas can freeze the wrong abstraction too early. |
| CI validation | Prevents broken or stale context from silently entering the repo. | CI integration and failure handling. | Too-strict checks may block normal development. |
| Source-of-truth precedence | Prevents stale docs from overriding code, schemas, migrations, or API contracts. | Repos need override rules. | Precedence can be wrong for unusual systems. |
| Provenance metadata | Lets agents and humans know where derived context came from. | Fingerprinting and dependency tracking. | False confidence if provenance is too coarse. |
| Trace output | Makes context loading, budget use, and execution path inspectable. | Trace schema and storage. | Traces may expose sensitive paths or data. |
| Stale-context policy | Defines when context is invalid, warning-only, or blocking. | Policy design and stale queues. | Teams may ignore warnings if not owned. |
| Trusted instruction review | Treats `tasks/` and `skills/` as privileged agent instructions. | CODEOWNERS and review gates. | Slower iteration on workflow files. |
| Basic threat model | Identifies prompt injection, malicious skills, stale context, and unsafe tool use. | Security review overhead. | Incomplete threat models can create false assurance. |
| Evaluation harness | Proves whether ALES improves task success, cost, and context size. | Benchmark design and maintenance. | Poor benchmarks can optimize the wrong behavior. |

### Required Before Enterprise Adoption

| Requirement | Why it matters | Complexity introduced | New risk |
|---|---|---|---|
| RBAC / least privilege | Controls which agents and users can read, write, refresh, or execute. | Identity integration. | Misconfigured roles can block work or permit too much. |
| Audit logging | Required for regulated or high-trust environments. | Durable event storage and retention policy. | Logs may contain sensitive information. |
| Secret and PII policy | Prevents context packets and traces from leaking protected data. | Classification and redaction. | Over-redaction may reduce agent usefulness. |
| Signed or trusted context | Prevents malicious context edits from steering agents. | Signing, verification, and key management. | Operational burden around keys and rotations. |
| Policy gates for tool use | Separates read-only, write, execution, and deployment privileges. | Runtime enforcement layer. | Too much friction can make users bypass the system. |
| Observability dashboards | Shows freshness, failures, cost, and evaluation trends. | Metrics pipeline and UI. | Teams may track vanity metrics instead of quality. |
| Tenant/project isolation | Required if ALES is hosted or spans multiple organizations. | Multi-tenant architecture. | Isolation bugs are severe. |

### Later Or Optional

- Multi-agent coordination.
- Distributed orchestration.
- Cross-organization task or skill marketplace.
- Formal certification.
- Advanced telemetry and optimization.
- Non-code domain production support.

These are not V1 blockers. Adding them too early would overcomplicate the system before the repo contract is proven.

## What Fails First At Scale

### 1. Freshness Drift

Context will go stale faster than the plan assumes. Code, tests, config, migrations, deployment rules, and team conventions change constantly.

Suggested mitigation:

- Track fingerprints for derived files.
- Run `ales validate` in CI.
- Maintain a stale queue with owners and severity.
- Auto-refresh low-risk generated map files.
- Open reviewable PRs for intent and skill changes.
- Make stale required context blocking only for high-risk tasks.

Tradeoff: this adds process overhead, but without it ALES becomes another documentation layer that agents trust too much.

### 2. Routing Mistakes

At scale, the agent may choose the wrong task, skill, module, service, or source of truth.

Suggested mitigation:

- Add a task registry.
- Add a machine-readable module or service index.
- Define source-of-truth precedence.
- Add eval queries that test routing quality.
- Track selected context in the trace.
- For monorepos, add ownership and dependency metadata.

Tradeoff: stronger routing requires more metadata and more validation, but it prevents agents from starting in the wrong part of the system.

### 3. Human Approval Bottleneck

If every context change requires human review, teams will stop maintaining ALES. If nothing requires review, ALES becomes unsafe.

Suggested mitigation:

- Classify context files by risk.
- Let generated map updates be auto-proposed.
- Require review for intent, tasks, skills, and privileged execution guidance.
- Use CODEOWNERS for domain-specific context.
- Keep human approval focused on judgment, not mechanical refreshes.

Tradeoff: risk-based review is more complex than a simple all-or-nothing rule, but it is the only approach likely to scale.

### 4. Security Through Context Files

`tasks/` and `skills/` are not ordinary docs. They are instructions that may steer agent behavior.

Suggested mitigation:

- Treat `tasks/` and `skills/` as privileged instruction surfaces.
- Require review before changes to privileged instructions are trusted.
- Separate advisory context from executable workflow instructions.
- Add least-privilege tool policies.
- Warn or block when untrusted context tries to request sensitive actions.

Tradeoff: this reduces flexibility, but it is required for any serious production setting.

### 5. Monorepo And Multi-Repo Complexity

Large systems will need more than a folder taxonomy. They need dependency information, ownership, service boundaries, generated clients, schemas, and runtime topology.

Suggested mitigation:

- Add project or module cards.
- Add dependency DAGs.
- Add bounded routing caps.
- Support incremental indexing.
- Prefer authoritative artifacts such as OpenAPI, AsyncAPI, protobuf, database migrations, package manifests, and CI configs.

Tradeoff: this moves ALES from a simple convention toward a real system, but large organizations will need that structure.

## Recommended Architecture Pattern

ALES should evolve into a layered architecture rather than only a folder convention.

### 1. Repo-Local ALES Contract

This is the human-visible contract stored in the repository:

- `ales.manifest.json`
- `intent/`
- `map/`
- `skills/`
- `tasks/`
- schemas
- provenance metadata

Problem solved: gives all agents a shared project-specific knowledge layer.

Complexity introduced: teams must maintain another structured layer.

New risk: stale or low-quality context can mislead agents unless validation exists.

### 2. Generated Machine Indexes

Generated indexes should be derived from authoritative project artifacts:

- source symbols
- OpenAPI or GraphQL specs
- AsyncAPI or event schemas
- protobuf files
- database migrations
- package manifests
- CI configuration
- test metadata
- deployment descriptors

Problem solved: avoids duplicating facts that already exist in machine-readable sources.

Complexity introduced: requires parsers, indexers, and artifact-specific logic.

New risk: generated indexes may appear authoritative even when the underlying parser misses semantic meaning.

### 3. Validator And CI Layer

The first production tool should be `ales validate`.

It should check:

- schema validity
- required files
- size limits
- stale fingerprints
- invalid provenance
- missing source-of-truth rules
- unreviewed privileged instruction changes
- broken task or skill references

Problem solved: prevents silent contract drift.

Complexity introduced: validation rules need configuration and migration support.

New risk: strict validation can slow teams if the failure modes are not carefully designed.

### 4. Runtime Context Packet Builder

The runtime should not dump all ALES files into the agent context. It should build a task-specific context packet.

A context packet should include:

- resolved task
- relevant skill
- selected map entries
- source evidence
- constraints
- expected output schema
- verification steps
- stale warnings
- token budget

Problem solved: turns the folder taxonomy into actual bounded context.

Complexity introduced: requires routing, ranking, dependency traversal, and budget logic.

New risk: incorrect packet construction can be worse than no packet because it gives the agent confident but incomplete context.

### 5. Policy And Permission Layer

ALES should define policy hooks even if V1 only implements a simple version.

Policies should control:

- read access
- write access
- command execution
- network access
- secret access
- generated file updates
- human approval requirements

Problem solved: prevents the context contract from becoming an unsafe automation surface.

Complexity introduced: policy enforcement depends on the runtime or adapter.

New risk: inconsistent policy enforcement across agents could fragment the ecosystem.

### 6. Trace And Evaluation Layer

Every meaningful run should emit a trace.

The trace should include:

- user request hash or summary
- resolved task
- selected skills
- loaded files
- skipped files and reasons
- stale warnings
- token budget estimate
- actions performed
- verification result
- output schema result

Problem solved: makes ALES behavior inspectable and measurable.

Complexity introduced: traces need schemas, storage, redaction, and review tooling.

New risk: traces can leak sensitive data if not redacted.

### 7. Refresh Workflow

Refresh should not silently rewrite human-owned context.

Recommended behavior:

- regenerate low-risk `map/` entries automatically or through PRs
- draft changes to `intent/` for human review
- require review for `tasks/` and `skills/`
- record refresh provenance
- keep stale status visible until merged

Problem solved: keeps context current without removing human ownership.

Complexity introduced: requires PR automation or equivalent review UX.

New risk: review queues can grow if ownership is unclear.

## Concrete Improvements

### Define Schemas First

Create schemas for:

- manifest
- task
- skill
- flow
- map entry
- provenance metadata
- stale status
- trace
- expected output

Why better: schemas turn ALES from prose into an implementable contract.

Complexity introduced: versioning and migration.

New risk: early schemas may be wrong and hard to change.

### Build `ales validate`

The first reference implementation should validate the contract before trying to automate everything.

Why better: validation catches drift early and gives teams confidence.

Complexity introduced: CLI design, CI integration, and config.

New risk: bad defaults could make validation noisy.

### Implement One Reference Task End To End

Pick one narrow software task, such as:

- add a REST endpoint
- debug a failing test
- add a database migration
- update an API contract

Why better: proves the loop with real artifacts instead of broad claims.

Complexity introduced: requires a real repo, baseline comparison, and trace capture.

New risk: one task may not generalize.

### Add Evaluation Queries

Define golden tasks and expected routing behavior.

Measure:

- task success
- files loaded
- token estimate
- clarification turns
- test pass rate
- stale context warnings
- human review burden

Why better: validates the core ALES claims.

Complexity introduced: benchmarks must be maintained as repos evolve.

New risk: teams may overfit to benchmark tasks.

### Define Source-Of-Truth Precedence

Default precedence for software repositories should be explicit.

Example:

```text
Runtime behavior / tests / API schemas / database migrations > production code > ADRs > architecture docs > README > agent-authored maps
```

Repos should be allowed to override this.

Why better: prevents stale documents from overriding authoritative artifacts.

Complexity introduced: not every repo has the same authority model.

New risk: bad precedence rules can hide important human intent.

### Add Trusted Instruction Controls

Changes to `tasks/` and `skills/` should require explicit trust.

Why better: prevents malicious or accidental instruction changes from steering agents.

Complexity introduced: CODEOWNERS, policy checks, and review rules.

New risk: extra friction may slow iteration.

### Narrow V1 To Software Repositories

Keep domain generalization as a research direction, not a V1 production claim.

Why better: makes validation realistic.

Complexity introduced: later non-code domains may need schema extensions.

New risk: narrower positioning may feel less ambitious, but it is more credible.

## Suggested V1 Implementation Sequence

1. Write JSON schemas for the core contract.
2. Build `ales validate` for local and CI use.
3. Add source-of-truth precedence to the manifest.
4. Add trusted-instruction rules for `tasks/` and `skills/`.
5. Implement one complete reference task with trace output.
6. Add benchmark/evaluation queries for that task.
7. Add refresh behavior for generated `map/` entries.
8. Add PR-based review for `intent/`, `tasks/`, and `skills/` changes.
9. Revisit broader domain generalization after software-repo validation.

## Decisions For V1

- ALES is a compatibility layer and knowledge contract for agents, not a replacement agent runtime.
- Skills describe workflow intent and project-specific rules; runtimes decide how to execute them with available tools.
- V1 needs validation, provenance, traceability, source-of-truth rules, and basic security controls.
- V1 does not need full enterprise RBAC, distributed orchestration, multi-agent coordination, or non-code production support.
- Claims about determinism, model-agnostic behavior, and reproducible outputs should be narrowed until benchmarks prove stronger claims.

## Open Questions To Resolve Next

1. What is the minimum schema set for the first reference implementation?
2. Should `ales validate` be a standalone CLI, a GitHub Action, or both?
3. What source-of-truth precedence should be the default for software repos?
4. Which reference task should be used for the first benchmark?
5. What changes to `tasks/` and `skills/` should be considered privileged?
6. How should stale context severity be represented?
7. What trace fields are required for V1 versus optional for later?
