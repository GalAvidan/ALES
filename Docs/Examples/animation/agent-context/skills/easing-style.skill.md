# Skill: Easing Style

## Purpose

All motion in this project uses a consistent easing curve unless a specific scene has a
`motion_override` in `map/scenes.json`. The agent must apply these defaults before emitting
any render spec.

## Default Easing

- **Curve:** ease-in-out cubic
- **Duration:** 0.3 seconds (unless scene metadata specifies a different duration)
- **Bezier:** `cubic-bezier(0.42, 0, 0.58, 1.0)`

## Per-Motion-Type Defaults

| Motion Type | In duration | Out duration | Notes |
|---|---|---|---|
| Element enter | 0.3s | — | Ease-in only on enter |
| Element exit | — | 0.2s | Ease-out only on exit |
| Camera move | 0.4s ease-in | 0.6s ease-out | Asymmetric: slower out |
| Logo reveal | 0.5s | 1.0s hold | No exit easing — holds until scene end |

## Overrides

- Scene 3 (product reveal) has `motion_override: slow-zoom` — use `cubic-bezier(0.3, 0, 1.0, 1.0)`
  with 0.3s ease-in and 1.0s ease-out. See `intent/decisions/slow-zoom-not-cut-scene3.md`.
- Any other scene-level override in `map/scenes.json` takes precedence over these defaults.
- Do not apply easing to audio fades — those are governed by `skills/audio-sync.skill.md`.
