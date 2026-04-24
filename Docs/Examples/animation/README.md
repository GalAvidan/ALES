# Example: Animation Project

A fully worked ALES `agent-context/` for a 30-second brand spot animation.

## What This Shows

- How `intent/` captures storyboard *intent* (emotional beats, why the slow zoom) separately from
  the timeline data in `map/scenes.json`.
- How `map/scenes.json` + `map/assets.json` create provenance: swapping an asset immediately
  identifies which scenes need re-rendering without searching the codebase.
- How `skills/` encode craft knowledge (easing curves, LUT application, audio alignment) that
  would otherwise produce inconsistent output.
- How `export-render` enforces project constraints (runtime ≤ 30.5s, product visible by t=12s)
  before emitting a render spec — the agent stops and asks rather than producing a non-compliant
  output.

## File Tree

```
agent-context/
├── intent/
│   ├── overview.md              ← deliverables, runtime constraints, core message
│   ├── storyboard.md            ← scene-by-scene emotional and visual intent
│   └── decisions/
│       └── slow-zoom-not-cut-scene3.md
│
├── map/
│   ├── scenes.json              ← all 5 scenes with timing, assets, audio sync points
│   ├── assets.json              ← all assets with paths and used_in_scenes links
│   └── flows/
│       └── scene-render.flow.json
│
├── skills/
│   ├── easing-style.skill.md
│   ├── color-grade.skill.md
│   └── audio-sync.skill.md
│
└── tasks/
    ├── add-scene.task.json
    ├── swap-asset.task.json
    └── export-render.task.json
```

## Key Agent Behaviors Enabled

| User says | Agent does |
|---|---|
| "Swap the product model to v4" | Resolves `swap-asset`, finds all scenes using `product-3d-model`, marks them stale, returns swap report for approval |
| "Add a teaser scene at the start" | Resolves `add-scene`, checks total runtime, warns if 30s limit would be exceeded, asks which scene to shorten |
| "Export the full spot" | Resolves `export-render`, checks for stale assets, runs `scene-render` flow for all scenes, verifies constraints, emits render spec |
