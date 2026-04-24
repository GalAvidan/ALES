# ALES — Domain Generalization

## Core Insight

The ALES four-layer taxonomy (`intent/`, `map/`, `skills/`, `tasks/`) is domain-agnostic at its core.
Any structured creative or analytical project has the same four knowledge planes — the only thing that changes is *what goes in each layer*.

---

## The Universal Mapping

| ALES Layer | In a Code Repo | In a Stats Presentation | In an Animation |
|---|---|---|---|
| **intent/** | Why this system is shaped this way | Audience, narrative arc, what story the data tells | Mood, style, storyboard intent, emotional beats |
| **map/** | Where modules live, what APIs exist | Which datasets, charts, slide sections exist and link to each other | Which scenes, assets, timelines, characters exist |
| **skills/** | How to add a feature in *this* repo | How to format *this project's* charts, citation style, brand palette | How to apply *this project's* easing, transitions, naming conventions |
| **tasks/** | Generic: "debug", "add-feature" | Generic: "add-slide", "update-data-source", "refresh-chart" | Generic: "add-scene", "sync-audio", "export-render" |

---

## What Makes the Agent "Smart" in Any Domain

The intelligence isn't in the agent model — it's in the **structured contract**:

1. **Provenance everywhere** — every claim in `/agent-context/` knows what source file it came from.
   When you swap a dataset or an asset, the agent knows *exactly* which slides or scenes are now stale.

2. **Bounded context** — the agent loads only what the task needs.
   For a stats presentation with 60 slides, it doesn't load all 60 when you ask it to fix slide 7.

3. **Skills = institutional knowledge** — the brand palette, the easing rules, the citation format.
   These are things that live in a human's head today. Capturing them in `skills/` makes the agent
   reproduce your taste, not average internet taste.

4. **Tasks = reusable procedures** — `add-slide`, `add-scene` are generic enough to port across
   projects, while skills make them project-specific.

5. **The staleness model still applies** — a dataset refresh triggers stale-tags on all charts derived
   from it. An asset update triggers stale-tags on all scenes using it. The agent detects lazily and
   resolves in batch.

---

## The Generalization Rule

> Any project where **a human has tacit knowledge that an agent lacks**, and where **outputs are
> derived from structured inputs**, benefits from this layer system.

That covers: code, data presentations, animations, design systems, research papers, video scripts,
game levels, legal documents, marketing campaigns — any domain where "what," "where," "how to do it
here," and "how to do it in general" are separable planes of knowledge.

---

## Open Question for Non-Code Domains

In code repos, `map/` is fully auto-derivable (parse files → extract symbols).
In creative projects, parts of `map/` require human judgment (e.g., "what is the narrative role of
slide 7?"). The **approval UX for intent drafts** (open question #4 from v1) becomes even more
critical here — the agent should draft, but humans must confirm narrative/intent-level map entries.

---

## Examples

- [`../Examples/stats-presentation/`](../Examples/stats-presentation/) — Q3 board deck with live data
- [`../Examples/animation/`](../Examples/animation/) — 30-second brand spot
