# Skill: Citation Style

## Purpose

Any data point shown in the deck that originates from an external source must be cited. This skill
defines the citation format so the agent produces consistent footnotes.

## When to Cite

- Any number sourced from a third-party report, government statistic, or market research firm.
- Internal data does **not** need a citation (it is assumed from our own systems).

## Format

Inline marker: superscript number in square brackets — `[1]`

Footnote block on the same slide (bottom, 8pt font):

```
[1] Source Name, "Report Title," Month Year. Retrieved YYYY-MM-DD.
```

## Examples

```
[1] Eurostat, "GDP Growth Rate — Euro Area," Q1 2026. Retrieved 2026-04-18.
[2] IDC, "European B2B SaaS Market Share Report," April 2026. Retrieved 2026-04-20.
```

## Rules

1. Do not cite internal datasets — only external sources.
2. If a slide has more than three citations, move the citation block to the appendix slide and add
   a footnote marker: `* See Appendix slide 17 for full references.`
3. Retrieval date is mandatory — use ISO 8601 (YYYY-MM-DD).
