# StrokeScope — on-device table-tennis stroke replay

A web app for table-tennis coaching sessions. It watches live camera video,
detects each stroke automatically, and plays back the last shots in slow motion
so a player and coach can study contact mechanics right at the table.

Everything runs on-device in the browser — no video ever leaves the phone.

## Features

- **Live camera capture** with an on-screen FPS guard (auto step-down 720p → 480p if the frame rate sags)
- **5-hit audio calibration** — tap once, let the app hear five normal ball contacts; no manual marking needed
- **Automatic stroke detection** — audio transient + motion, gated by MediaPipe pose (wrist-velocity spike + elbow sweep)
- **Manual MARK SHOT** button as backup and as ground truth for the detection report
- **Track-player workflow** — tap *Track player*, tap the player once; auto-follow framing keeps them centered
- **Replay** — latest shot plus last-four reel, at 1×, ½×, ⅕×, with frame stepping and a contact loop
- **Detection report** — compares automatic marks against kept manual taps within ±500 ms
- **Session export** — `stroke-replay-session-log/v1.3` JSON for offline analysis

## Run it

Camera access requires a secure context: serve the folder over HTTPS (or
`localhost`) and open `index.html`. The MediaPipe pose model loads from CDN at
runtime.

## Project log

`CHANGELOG.md` tracks the build history. `dfmea-stroke-detection.md` is the
design FMEA for the detection pipeline (12 failure modes, RPN-ordered).

## Privacy

No backend, no uploads, no analytics. The pose model and all detection run
locally; covering the camera provably stops all processing. Note: the pose
model file itself loads from the jsDelivr CDN at startup (see the planned
security review in `CHANGELOG.md`).

## Known limitations

- Pose tracking needs the player flagged first (*Track player*, then tap them);
  it works best with the rear camera, side-on to the table.
- Detection is assistive: manual marks are the ground truth, and the detection
  report calls unmatched auto-detections "unmatched", never false alarms.
- In low light the fps guard steps capture 720p → 480p to protect timing; add
  light when you can.
- Long sessions grow the exported log (~10 Hz telemetry) — exports stay a few
  MB for a typical session.
