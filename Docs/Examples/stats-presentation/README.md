# Example: Statistics Presentation

A fully worked ALES `agent-context/` for a Q3 board deck with live data sources.

## What This Shows

- How `intent/` captures narrative arc and audience rules that a generic LLM has no way to know.
- How `map/slides.json` + `map/datasets.json` create provenance links between charts and their data,
  so the agent knows *exactly* which slides go stale when a dataset is refreshed.
- How `skills/` encode institutional knowledge (brand colors, number format, citation style) that
  would otherwise produce inconsistent output.
- How tasks like `refresh-chart` and `add-slide` are generic procedures that become project-smart
  by loading the right layers.

## File Tree

```
agent-context/
├── intent/
│   ├── overview.md              ← deck purpose, constraints
│   ├── audience.md              ← who's in the room and what they need
│   ├── narrative.md             ← slide-by-slide story arc
│   └── decisions/
│       └── bar-not-pie-slide7.md
│
├── map/
│   ├── slides.json              ← all 20 slides with data source links
│   ├── datasets.json            ← all datasets with paths and used_by_slides
│   └── flows/
│       └── data-to-chart.flow.json
│
├── skills/
│   ├── brand-colors.skill.md
│   ├── number-formatting.skill.md
│   └── citation-style.skill.md
│
└── tasks/
    ├── refresh-chart.task.json
    ├── add-slide.task.json
    └── proofread-narrative.task.json
```

## Key Agent Behaviors Enabled

| User says | Agent does |
|---|---|
| "Update the revenue chart" | Resolves `refresh-chart`, loads only `slide-03` + `revenue-quarterly` dataset, applies brand colors + number format, emits new chart spec |
| "Add a slide about churn" | Resolves `add-slide`, checks slide count vs 20-slide limit, validates narrative fit, drafts entry, asks for approval |
| "Proofread the deck" | Resolves `proofread-narrative`, loads narrative arc + audience rules, checks all slide titles, flags jargon and missing deltas |
