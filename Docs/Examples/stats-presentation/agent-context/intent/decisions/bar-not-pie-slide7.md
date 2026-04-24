# Decision: Bar Chart for Slide 7 (New Customer Cohort)

**Date:** 2026-04-20
**Status:** Decided

## Context

Slide 7 shows the B2B new-customer cohort count by quarter (Q1–Q3). Two chart types were considered:
bar chart and pie chart.

## Decision

Use a **grouped bar chart** (not a pie chart).

## Rationale

- The audience needs to compare cohort size *across time* (Q1 vs Q2 vs Q3), which bars do well.
- Pie charts encode part-of-whole relationships; cohort count is not a share of a total — it is an
  absolute count. A pie here would be semantically wrong.
- Grouped bars also let us overlay the plan line as a secondary axis if needed in Q4 updates.

## Consequences

- All quarterly cohort charts in this deck use bars for consistency.
- If a "share of revenue by segment" metric is added, that slide may use a pie (different question).
