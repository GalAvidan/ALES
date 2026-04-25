# ALES V3 — Tools & MCP Integration Plan

## Where This Fits in the ALES Roadmap

| Version | Theme | What it added |
|---|---|---|
| **V1** | Core contract | Four-layer taxonomy, staleness model, action vocabulary, execution loop |
| **V2** | Generalization | Proved the four layers work for any domain (code, animation, data) |
| **V3** | External reach | Gives agents a governed way to call external capabilities via MCP |

V3 does not replace or restructure V1/V2. It adds a **fifth layer** (`tools/`) and one new action verb (`invoke_tool`) to the closed vocabulary.

---

## The Problem V3 Solves

The V1 action vocabulary (`load`, `search`, `invoke_skill`, `emit`, `ask`, …) is entirely **read-side and text-side**. It can reason about a project, but it cannot:

- Generate an image, render a frame, or run a test suite
- Query a live database or external API
- Push output to a downstream system (CDN, Figma, Slack, etc.)

MCP (Model Context Protocol) is the standard wire format for agent-to-tool communication. ALES's job is not to re-invent that protocol — it is to provide the **project-level governance layer on top of it**: when to call a tool, what constraints apply, how to inject project skills into the call, and how to register the output back into the map.

---

## V3 Core Idea: The `tools/` Layer

```
/agent-context/
├── intent/      ← WHY (human-authored)
├── map/         ← WHAT EXISTS (agent-derived)
├── skills/      ← HOW TO DO IT HERE (project-specific recipes)
├── tasks/       ← HOW TO EXECUTE (generic procedures)
└── tools/       ← WHAT EXTERNAL CAPABILITIES ARE AVAILABLE  ← NEW
    └── *.tool.json
```

A `*.tool.json` file is the bridge between ALES and an MCP server. It contains three things:

1. **MCP coordinates** — where the server lives and what its tool name is
2. **Project governance** — constraints, skill injections, when-to-call / when-NOT-to-call rules
3. **Cost profile** — latency and token overhead, so the task budget model stays accurate

---

## The `*.tool.json` Schema

```json
{
  "tool_id": "string — unique within this project (e.g. 'generate-thumbnail')",
  "version": "semver string",
  "description": "string — one sentence, what this tool does",

  "mcp": {
    "server_uri": "string — base URI of the MCP server",
    "tool_name":  "string — the tool name as declared by the MCP server",
    "transport":  "string — 'http/sse' | 'stdio' | 'ws'"
  },

  "inputs": {
    "<param_name>": "string — type and description of the parameter"
  },

  "outputs": {
    "<field_name>": "string — type and description of the returned value"
  },

  "project_guidance": {
    "always_inject_skills": [
      "array of skill paths to auto-include in every call to this tool"
    ],
    "constraints": [
      "array of hard rules the agent must check before calling"
    ],
    "when_to_call": [
      "array of conditions that make this tool the right choice"
    ],
    "when_NOT_to_call": [
      "array of conditions that should prevent calling this tool"
    ]
  },

  "cost_profile": {
    "latency":       "string — expected latency range",
    "token_overhead": "number — estimated tokens consumed by this call",
    "billing_unit":  "string — 'per-image' | 'per-request' | 'per-token' | etc."
  }
}
```

### Why these fields?

| Field group | Purpose in ALES |
|---|---|
| `mcp` | The only part that talks to the vendor. Swapping a provider = changing this block only. |
| `inputs` / `outputs` | Lets the task step declare what it passes in and what it registers back. |
| `project_guidance` | Where ALES's value lives. Vendor knows nothing about your brand; this file does. |
| `cost_profile` | Feeds the task `token_budget` model — a tool call with 200 token overhead must be accounted for. |

---

## A Concrete Example: `generate-thumbnail.tool.json`

```json
{
  "tool_id": "generate-thumbnail",
  "version": "1.0.0",
  "description": "Generate a still thumbnail image for a scene or slide using the project's visual identity.",

  "mcp": {
    "server_uri": "https://image-gen.acme-tools.io/mcp",
    "tool_name":  "generate_image",
    "transport":  "http/sse"
  },

  "inputs": {
    "prompt":    "string — describe what to generate (auto-built from scene/slide context)",
    "aspect":    "string — '16:9' | '1:1' | '9:16'",
    "style_ref": "string — inline style instructions derived from brand-colors skill"
  },

  "outputs": {
    "image_url": "string — CDN URL of the generated image",
    "asset_id":  "string — ID to register in map/assets.json"
  },

  "project_guidance": {
    "always_inject_skills": [
      "skills/brand-colors.skill.md",
      "skills/color-grade.skill.md"
    ],
    "constraints": [
      "Never use aspect ratio 4:3 — violates brand guidelines",
      "Generated images must be registered in map/assets.json before use in any scene"
    ],
    "when_to_call": [
      "A task step has action: invoke_tool and tool: generate-thumbnail",
      "An asset slot in map/assets.json has type: placeholder"
    ],
    "when_NOT_to_call": [
      "A real (non-placeholder) asset already exists for this slot",
      "The scene referencing this asset is marked status: locked"
    ]
  },

  "cost_profile": {
    "latency":        "3–8 seconds",
    "token_overhead": 200,
    "billing_unit":   "per-image"
  }
}
```

---

## The New Action Verb: `invoke_tool`

V3 adds exactly one word to the V1 closed action vocabulary:

```
invoke_tool  →  call an external tool via MCP, governed by a *.tool.json file
```

All existing verbs are unchanged.

### How it looks in a task step

```json
{
  "step": 3,
  "action": "invoke_tool",
  "tool": "generate-thumbnail",
  "inputs": {
    "prompt":    "derived from scene description in map/scenes.json",
    "aspect":    "16:9",
    "style_ref": "derived from skills/brand-colors.skill.md"
  },
  "on_success": {
    "action": "mark_map",
    "target": "map/assets.json",
    "update": "register returned asset_id, set stale: false"
  },
  "purpose": "Generate a real thumbnail for any scene with asset type: placeholder"
}
```

### How it looks in a flow step

Flows can use `invoke_tool` in exactly the same way, sitting alongside `invoke_skill`:

```json
{
  "step": 3,
  "action": "invoke_tool",
  "tool":   "figma-export",
  "purpose": "Pull the latest Figma frame for this slide before generating the chart spec"
}
```

---

## The Execution Path for `invoke_tool`

```
Task step fires: action = "invoke_tool", tool = "generate-thumbnail"
        │
        ▼
1. AGENT loads agent-context/tools/generate-thumbnail.tool.json

2. AGENT checks project_guidance.when_NOT_to_call
   └─ if any condition is true → STOP, do not call, explain why

3. AGENT checks project_guidance.constraints
   └─ if any constraint would be violated → STOP, ask human

4. AGENT loads each skill in project_guidance.always_inject_skills
   └─ uses skill content to enrich the prompt / call parameters

5. AGENT adds cost_profile.token_overhead to running token counter
   └─ if this would exceed task token_budget → ask human before proceeding

6. MCP CLIENT opens connection:
     server_uri: https://image-gen.acme-tools.io/mcp
     tool_name:  generate_image
     transport:  http/sse

7. MCP SERVER executes → returns { image_url, asset_id }

8. AGENT runs on_success action:
   └─ registers asset in map/assets.json, sets stale: false

9. Task continues to next step
```

---

## Staleness: How Tools Fit the Existing Model

Tool calls produce **outputs that land in the map** (assets, datasets, rendered files). The existing staleness model already handles these:

| Trigger | Result |
|---|---|
| `*.tool.json` `server_uri` changes | All map entries produced by this tool should be re-verified |
| Map entry `stale: true` for a tool-produced asset | Next `invoke_tool` call regenerates it |
| Tool call fails | Map entry is written with `stale: true` and `stale_reason: "tool call failed"` |
| Project skill changes (e.g. brand colors updated) | All tool-produced assets derived from that skill should be marked stale |

No new staleness mechanism is needed. The `mark_stale` verb and `_meta` header pattern from V1 cover all these cases.

---

## Tool Discovery

How does an agent know which tools are available?

**Option A — Index file (recommended for V3)**

A `tools/_index.json` file lists all available tools with a one-line description:

```json
{
  "tools": [
    { "tool_id": "generate-thumbnail", "description": "Generate still images via MCP image server" },
    { "tool_id": "figma-export",       "description": "Pull latest frame from Figma via MCP" },
    { "tool_id": "run-tests",          "description": "Execute the test suite via local MCP stdio server" }
  ]
}
```

The agent loads `tools/_index.json` at PLAN time (step 2 of the execution loop), the same way it already loads task definitions. It then loads the specific `*.tool.json` only when the task step requires it — preserving bounded context.

**Option B — Inline in task**

For tasks that always use a specific tool, the task file can name the tool explicitly in its step (`tool: "generate-thumbnail"`). No index lookup needed. The agent loads the `.tool.json` directly.

Both options are compatible. Index is useful for agent-initiated tool selection; inline is useful for deterministic task flows.

---

## Where Tools Live in the Folder Structure

```
/agent-context/
├── intent/
│   └── overview.md
├── map/
│   ├── scenes.json       ← or slides.json, modules.json, etc.
│   ├── assets.json
│   └── flows/
│       └── scene-render.flow.json
├── skills/
│   ├── brand-colors.skill.md
│   └── easing-style.skill.md
├── tasks/
│   ├── export-render.task.json
│   └── add-scene.task.json
└── tools/                          ← NEW
    ├── _index.json                 ← discovery manifest
    ├── generate-thumbnail.tool.json
    ├── figma-export.tool.json
    └── run-tests.tool.json
```

---

## Updated ALES Layer Table

| Layer | Question it answers | Owner | Freshness model |
|---|---|---|---|
| **intent/** | *Why* the project is shaped this way | Human | Manual; rarely stale |
| **map/** | *What* exists and how things relate | Agent (derived) | TTL + stale-tag |
| **skills/** | *How to do recurring work in this project* | Human (agent suggests) | Versioned |
| **tasks/** | *How an agent executes a class of work* | Spec / shared | Spec-versioned |
| **tools/** | *What external capabilities are reachable, and under what rules* | Human + vendor | Stale when server_uri or skills change |

---

## Updated Closed Action Vocabulary

```
load            → read file(s) into context
search          → scoped search
identify_flow   → match task to flow in map/flows/
follow_flow     → traverse a flow's path
inspect_code    → read symbols within loaded files
invoke_skill    → apply a project-specific skill
invoke_tool     → call an external tool via MCP  ← NEW in V3
mark_stale      → tag a context file as stale
emit            → produce structured output
ask             → request user clarification (terminates step)
```

---

## What V3 Does NOT Change

- The four-layer taxonomy (tools is additive, not a replacement)
- The staleness model (tool outputs are just map entries)
- The execution loop (RESOLVE → PLAN → LOAD-HIGH → EXECUTE → VERIFY → EMIT)
- The token budget model (`token_overhead` in the tool file feeds the same counter)
- The `ask` guardrail (constraint violations and budget overruns still stop the agent)

---

## V3 Non-Goals (Deferred to V4+)

- **Multi-tool orchestration** — chaining two MCP tools within a single step
- **Agent-initiated tool discovery** — agent queries an MCP registry to find new tools at runtime
- **Write-back to external systems** — tools that push output to a repo, CDN, or database (V3 tools return data; the agent decides what to register)
- **Tool versioning / rollback** — pinning a specific MCP server version
- **Cross-project shared tool registries** — a single `tools/` folder shared across multiple ALES projects

---

## Open Questions for V3

1. **Auth model** — How does the agent obtain credentials for the MCP server? Options: environment variable, a `secrets/` layer outside agent-context, or a vendor-supplied token injected at runtime. The `.tool.json` file must never contain credentials.

2. **Retry and fallback** — If an MCP call fails transiently, should the tool file declare a retry policy, or is that always task-level?

3. **No-tool-found fallback** — Same open question as V1's "no-skill-found" but for tools. Does the agent ask, or skip the step?

4. **Tool testing** — How do we verify a `*.tool.json` is correctly wired before a task depends on it? A `ping-tool` task that does a dry-run call?

---

## Example: Updating an Existing Task to Use a Tool

The existing `export-render.task.json` currently emits a render *specification*. With V3, it could invoke a real render tool:

**Before (V1/V2):**
```json
{
  "step": 7,
  "action": "emit",
  "output_schema": "render-spec",
  "purpose": "Return the full render specification for the pipeline"
}
```

**After (V3):**
```json
{
  "step": 7,
  "action": "invoke_tool",
  "tool": "render-engine",
  "inputs": {
    "spec": "the render-spec produced in prior steps",
    "format": "derived from task input"
  },
  "on_success": {
    "action": "mark_map",
    "target": "map/assets.json",
    "update": "register returned render output URL, set stale: false"
  },
  "purpose": "Submit the render spec to the render engine and register the output"
},
{
  "step": 8,
  "action": "emit",
  "output_schema": "render-result",
  "purpose": "Return the render output URL and metadata"
}
```

The task now goes all the way to a real artifact instead of stopping at a specification. Everything upstream (intent constraints, skill injection, budget tracking) still applies unchanged.
