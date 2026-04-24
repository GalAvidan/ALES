# Decision: Slow Zoom on Scene 3 (Product Reveal)

**Date:** 2026-04-21
**Status:** Decided

## Context

Scene 3 is the product reveal — the most important moment in the spot. Two motion options were
considered: a hard cut directly to the product, or a slow zoom emerging from the transition particle.

## Decision

Use a **slow zoom** (0.3s ease-in, 1.0s ease-out, cubic bezier), not a cut.

## Rationale

- A hard cut at this moment would feel abrupt and waste the tension built in Scene 2. The viewer
  hasn't had time to process the transition.
- The slow zoom lets the product "earn" its entrance — it emerges naturally from the single particle
  of light, maintaining visual continuity.
- Test audiences in the concept phase responded more positively to the zoom variant (7/10 vs 4/10
  for the cut variant on "felt satisfying").

## Consequences

- Scene 3 duration is 8 seconds (not 6) to accommodate the zoom. Scene 4 is correspondingly
  shorter at 8 seconds.
- The easing on this zoom overrides the default easing in `skills/easing-style.skill.md`.
  This is the only scene with a non-default easing curve.
- If the runtime constraint tightens, this scene is the first candidate for trimming — but only
  after re-testing with audiences.
