# Skill: Brand Colors

## Purpose

Ensure every chart in this deck uses the approved brand palette. The agent must apply these colors
before emitting any chart specification.

## Primary Palette

| Role | Hex | Usage |
|---|---|---|
| Primary Blue | `#0057B8` | Main data series, bars, lines |
| Accent Orange | `#FF6B00` | Highlight / call-out series (e.g., plan line, current quarter) |
| Neutral Grey | `#8C8C8C` | Secondary / comparison series |
| Success Green | `#00875A` | Positive delta, above-plan indicators |
| Warning Red | `#D63B2F` | Negative delta, below-plan indicators |

## Background / Text

| Role | Hex |
|---|---|
| Slide background | `#FFFFFF` |
| Body text | `#1A1A1A` |
| Axis labels | `#555555` |
| Gridlines | `#E0E0E0` |

## Rules

1. Never use more than three series colors on a single chart.
2. The plan line (when shown) is always Accent Orange, dashed, 1.5 pt.
3. Do not use brand colors for decorative elements — only for data-encoding.
4. When a chart shows a positive/negative split (e.g., waterfall), use Success Green for gains and
   Warning Red for losses. Primary Blue is not used on waterfall charts.
