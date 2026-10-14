# StrokeScope — Changelog

All notable changes to the app, newest first. Dates are Pacific.

## Unreleased (planned)

- Security review pass: SRI-pin the MediaPipe CDN script, evaluate self-hosting
  the pose model files and the Google Fonts, confirm nothing phones home.

## 2026-10-12 — v2: pose auto-marking + detection report

- Renamed to **StrokeScope** (user's pick).
- **Pose-based auto-marking:** tap "Track player," tap the player once, and the
  on-device MediaPipe pose model marks strokes from the wrist-velocity spike
  plus elbow sweep inside a ~450 ms window. Neighboring tables are ignored.
  Manual MARK stays as the backup/override.
- **Auto-follow framing:** once tracked, a smoothed crop follows the player
  side to side (deadband so small sways don't jitter); falls back to full
  frame if the player is lost. Applies to stored clips, not just preview.
- **Detection report panel:** manual taps are ground truth; every auto
  detection matches a tap within ±500 ms. Per detector (pose-gated,
  audio+motion) it shows caught count, recall on tapped shots, misses with
  timestamps, and "unmatched" (never "false"). Recomputes live as you tap.
  Exported log schema is now v1.3.
- Third field-test log analyzed (169 s): 14/14 manual marks kept, 2 auto marks
  kept via audio+motion coincidence, tap blanking fixed, calibration threshold
  landed ~4.4. Snappiness traced to the camera delivering 14.7 fps effective
  (~68 ms between frames) against 60 requested.

## 2026-10-11 — fps guard

- Live fps pill on the video (turns red under 24 fps).
- Auto step-down from 720p to 480p after 3 slow seconds, with an on-screen
  banner; plain "low light" hint if still slow. Guard state recorded in the
  exported log.
- Automated checks can't exercise a real camera — the next pasted session log
  is the proof.

## 2026-10-08/09 — detection retune + playback upgrades

- Relative audio threshold via 5-hit calibration (replaces the fixed 22 that
  sat far above real 5–9 hit transients), 200 ms tap blanking, snap-to-onset
  marking. Log schema v1.1.
- Playback: frame-step, loop-around-contact replay, 60 fps capture attempt,
  scrub bar.
- Second field-test log analyzed (101 s): 9/9 manual marks kept, 0 auto kept —
  the data that drove the retune.

## 2026-10-08 — session-log export instrumentation

- Export session log: compact JSON of detector firings, confidence scores,
  manual marks and flags, ~10 Hz audio/motion telemetry. Numbers only — no
  video, no pose frames. Designed for paste-back diagnosis.
- Per-clip download buttons for marked shots.
- First field test ran (video recorded, replay tried; on-device only, no
  telemetry by design).

## 2026-10-06 — v1

- Live capture with auto/manual shot marking (audio onset + motion-region
  coincidence), last-1 / last-4 clip replay at 1×/½×/⅕×, 5-hit calibration,
  noisy-hall mode (manual marks lead), everything on-device.
- DFMEA: 12 failure modes, saved with the project. Follow-on rundown of
  FMECA, FTA, STPA, DVP&R, 8D delivered as a companion doc.
