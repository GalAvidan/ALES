# Skill: Audio Sync

## Purpose

Animation keyframes must align to the audio beat markers declared in `map/scenes.json`. This skill
defines how to perform the alignment.

## Sync Points

Each scene in `map/scenes.json` declares `audio_sync_points` — a list of `{ beat, time_sec }`
entries. The agent must:

1. Identify the animation keyframe that corresponds to each beat's visual event.
2. Shift or stretch the keyframe timing to land exactly on the declared `time_sec`.
3. Re-apply easing (per `skills/easing-style.skill.md`) after any timing adjustment.

## Beat Types and Their Visual Counterparts

| Beat name | Visual event |
|---|---|
| `beat 1..N` | Hard cut or element enter |
| `drop` | Screen transition completes (Scene 2 end) |
| `product-visible` | Product fully readable in frame (Scene 3) |
| `action` | User action begins in demo (Scene 4) |
| `outcome` | Result visible in demo (Scene 4) |
| `logo-in` | Logo begins fade-in (Scene 5) |

## Tolerance

- Keyframe must land within ±2 frames of the declared `time_sec` at the target frame rate.
- At 60fps: ±33ms. At 30fps: ±67ms.
- If alignment within tolerance is not possible without distorting the motion, flag with `ask`
  and propose a timing adjustment to the audio.

## Do Not

- Do not shift audio to match animation. Audio is the source of truth.
- Do not add or remove frames to meet timing. Adjust keyframe positions only.
