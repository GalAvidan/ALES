# Skill: Number Formatting

## Purpose

All numbers displayed in this deck — on charts, in callout boxes, and in slide body text — must
follow these rules exactly. The agent must apply these rules before emitting any chart spec or
slide content.

## Currency

- Currency: **EUR (€)**
- Symbol position: prefix, no space (`€1.2M` not `1.2M €`)
- Thousands separator: `.` (German convention — `€1.234.567`)
- Decimal separator: `,` (German convention — `€1,2M`)

## Scale Abbreviations

| Scale | Abbreviation | Example |
|---|---|---|
| Millions | M | `€12,3M` |
| Billions | B | `€1,2B` |

- Always abbreviate when the value ≥ 1 000 000.
- Never show raw thousands in chart labels (use `€1,2M` not `€1.200.000`).

## Decimal Places

- **Chart axis labels:** 0 decimal places (e.g., `€12M`)
- **Chart callout / data labels:** 1 decimal place (e.g., `€12,3M`)
- **Percentage values:** 1 decimal place (e.g., `12,3%`)
- **Per-unit costs:** 2 decimal places (e.g., `€0,42`)

## Zero Values

- Never display a bare `0` or `€0`. Use `—` (em dash) in tables and suppress the data label on
  charts.

## Negative Numbers

- Use minus sign prefix: `−€1,2M` (Unicode minus U+2212, not hyphen).
- Never use parentheses for negatives in charts.
