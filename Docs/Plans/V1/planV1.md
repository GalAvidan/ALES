# ALES v1 — Design Summary (so far)

## 🎯 Core Thesis

> ALES is a repository-side execution contract that lets any LLM agent complete repo-bound tasks with **bounded context, deterministic traversal, and reproducible outputs** — independent of the underlying model.

Three testable properties:
- **Bounded context** (measurable in tokens)
- **Deterministic traversal** (same task + repo state ⇒ same file load order)
- **Model-agnostic** (works on any capable LLM)

---

## 🧠 Mental Model

`/agent-context/` is **not a code index**. It is the answer to:

> *"What would an experienced engineer on this project tell a new hire?"*

This means most of the cache stays valid even when code churns daily — only architectural / API-shape / intent changes invalidate it.

---

## 🏛️ Two Truths

| Truth | Role |
|---|---|
| **Code** | Ground truth. Always wins on factual conflict. |
| **Intent** | Human-owned context. Code can't contradict it because they describe different things (why vs. what). |

---

## 📐 Four-Layer Taxonomy

| Layer | Question it answers | Owner | Freshness model |
|---|---|---|---|
| **intent/** | *Why* and *how* the system is shaped | Human (agent drafts) | Manual; rarely stale |
| **map/** | *Where* things live, *what* exists | Agent (derived) | TTL + stale-tag → batch refresh |
| **skills/** | *How to do specific recurring tasks in THIS repo* | Human (agent suggests) | Versioned; stale only if intent changes |
| **tasks/** | *How an agent executes a generic class of work* (debug, add feature, refresh) | Spec / shared | Spec-versioned |

**Key distinctions:**
- **Task** = generic procedure, portable across repos
- **Skill** = project-specific recipe (assumes agent already has general capabilities like git, file I/O)
- A task may *invoke* a skill

---

## 📁 Folder Structure

```
/agent-context/
├── ales.manifest.json         ← spec version, schemas, repo SHA
│
├── intent/                    ← authored, slow-changing
│   ├── overview.md
│   ├── architecture.md
│   ├── conventions.md
│   ├── glossary.md
│   └── decisions/             ← ADR-style "why we chose X"
│
├── map/                       ← derived, agent-maintained
│   ├── modules.json
│   ├── apis.json
│   ├── flows/
│   └── _provenance.json
│
├── skills/                    ← project-specific recipes
│   └── *.skill.md
│
└── tasks/                     ← agent execution procedures
    └── *.task.json
```

---

## 🔄 Staleness Model — "Lazy detect, batch refresh"

**Every derived/drafted file carries a header:**

```json
{
  "_meta": {
    "generated_at": "2026-04-15T10:00:00Z",
    "ttl_days": 7,
    "stale": false,
    "stale_reason": null,
    "derived_from": ["src/Orders/**"],
    "fingerprint": "sha256:..."
  }
}
```

**Two phases, deliberately separated:**

1. **Detection (continuous, distributed, cheap)**
   Any agent, mid-task, can call a `mark-file-as-stale` skill when it notices drift. One-line write. No refresh yet.

2. **Refresh (deliberate, batched, out-of-band)**
   Triggered explicitly (user, CI, or a `refresh` task). Sweeps the repo for entries where:
   - `stale == true`, OR
   - `now - generated_at > ttl_days`, OR
   - `fingerprint` mismatches `derived_from`

   Then:
   - **map/** entries → agent re-derives autonomously
   - **intent/** entries → agent drafts diff, requests human approval

**Property:**
> Staleness is detected continuously and lazily, but resolved deliberately and in batch.

---

## ⚙️ Execution Loop (v1.1)

```
1. RESOLVE     → match user request → task definition
2. PLAN        → load task; compute step DAG; estimate token budget
3. LOAD-HIGH   → load only high-priority context
4. EXECUTE     → run steps in order
   ├─ on missing context → escalate priority
   ├─ on ambiguous flow  → STOP, ask (no guessing)
   └─ on budget exceeded → emit partial + reason
5. VERIFY      → check output against expected_output schema
6. EMIT        → return result + trace (files loaded, steps run, tokens used)
```

**Mandatory guarantees:**
- No silent context expansion beyond declared priorities
- Trace is mandatory (makes "deterministic" verifiable)
- Budgets declared per task

---

## 🔤 Closed Action Vocabulary (v1)

```
load            → read file(s) into context
search          → scoped search
identify_flow   → match task to flow in map/flows/
follow_flow     → traverse a flow's path
inspect_code    → read symbols within loaded files
invoke_skill    → apply a project-specific skill
mark_stale      → tag a context file as stale
emit            → produce structured output
ask             → request user clarification (terminates step)
```

Anything outside this set = v2.

---

## 🧬 Provenance Everywhere

> Every claim in `/agent-context/` is traceable to the source files it was derived from, and self-invalidates when those files change.

This is the spec's defining property and a key paper contribution.

---

## 🚫 Explicit v1 Non-Goals

- Auto-generation of map without agent involvement
- Multi-agent coordination
- In-band (mid-task) refresh — refresh is always out-of-band
- Runtime learning / self-updating skills
- Cross-repo flows
- Write-side actions beyond context maintenance

---

## ❓ Still Open

1. **Skill discovery mechanism** — name match? tags? agent reads index?
2. **No-skill-found fallback** — generic task flow? auto-propose new skill (v2)?
3. **First-contact bootstrap** — is "bootstrap" itself a task in `tasks/`?
4. **Approval UX for intent drafts** — PR? `_pending/` folder? CLI prompt?

---

## ✅ Ready For

- JSON schemas (manifest, task, flow, skill, _meta)
- Reference loader (~200 LOC)
- POC against one real repo (pizza example?)
- *Then* benchmark protocol

---

Want to tackle the 4 open questions next, or jump to picking the POC repo / drafting the first schema?