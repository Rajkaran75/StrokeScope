# DFMEA — Stroke Detection Subsystem

**Headline:** In a quiet hall the app "just works" on sound; in a real hall, sound is a vote, not a verdict. Requiring audio+motion coincidence (or pose-tracked swing kinematics) plus the coach's thumb as ground truth is what makes detection trustworthy. The fixes below are already in the v1 build.

**Scope:** Deciding "a stroke by the filmed player happened at time T" in a crowded training hall, on a phone browser, on-device. Ratings are 1–10 engineering judgment; RPN = Severity × Occurrence × Detection. Sorted worst first. Analysis date: 2026-10-06.

## Rating scale (1–10)

- **Severity (S):** 1 = cosmetic, no coaching impact … 10 = tool teaches wrong form or session data is lost
- **Occurrence (O):** 1 = essentially never … 10 = expected nearly every rally
- **Detection (D):** 1 = immediately obvious at setup … 10 = silent failure, discovered only later

## Failure modes

| # | Failure mode | S | O | D | RPN | Effect | Fix in v1 |
|---|--------------|---|---|---|-----|--------|-----------|
| 1 | Soft shots missed entirely | 7 | 6 | 6 | **252** | The threshold raised to reject neighboring tables also eats drop shots and short pushes. The exact stroke the coach wanted is gone, silently — nothing flags a miss. | Per-session calibration (tap along with 5 hits to learn your table's level); once audio+motion coincidence is required, the audio threshold can drop because motion carries half the decision. |
| 2 | Coach forgets to tap mid-rally | 6 | 6 | 7 | **252** | Marking every shot by hand in a fast rally is cognitive overload; thumbs stop. Gaps in the shot history, noticed only later. | Auto-detect never turns off — it runs as backfill; the final shot list is the union of manual + auto marks. |
| 3 | Neighboring table's hit logged as ours | 8 | 7 | 4 | **224** | One mic, no direction finding, loud hall. Replay shows the wrong player's strokes — the tool teaches wrong form, which is worse than useless. | (a) Require audio+motion coincidence — sound counts only if your player is visibly swinging within ±150 ms; (b) table-mic mode — a second device (or wired earbud mic) on your table isolates your hits; (c) noisy-hall mode demotes auto to backfill, thumb becomes primary. Pose-gated visual is the stronger version: track the filmed player's arm skeleton, ignore everyone else's. |
| 4 | Filming nearby players without consent | 6 | 5 | 6 | **180** | Usage hazard, not a software bug, but DFMEA covers it. | Framing guide overlay; on-device-processing notice; first-run venue-permission reminder. All video stays on the device — nothing uploads. |
| 5 | Coach taps late | 5 | 8 | 4 | **160** | Human reaction time is 300–500 ms, so manual marks land late and the contact moment sits at the clip's edge or outside it. | Back-date manual marks by ~350 ms automatically; keep 1.5 s pre-roll so the stroke is always inside the clip. |
| 6 | Phone overheats / browser kills the page mid-session | 8 | 4 | 5 | **160** | An hour of video encode in a warm hall. Dead app, lost session. | Cap capture at 720p30; hold a wake lock; show a low-power path. |
| 7 | Browser audio processing eats the transients | 7 | 5 | 4 | **140** | getUserMedia defaults (echo cancellation, noise suppression) are designed to *remove* exactly the sharp pop we're listening for — auto-detect goes silently deaf. | Request raw mic (all processing off); show a live detection meter so the coach sees the app is hearing. |
| 8 | Audio/video clock drift misaligns clips | 6 | 4 | 5 | **120** | Two pipelines, two clocks; the clip window slides off the contact moment. | Stamp both pipelines from the same clock; keep the clip window wide. |
| 9 | Shouting and clapping false-trigger | 4 | 8 | 3 | **96** | "Cho!" after every point is a broadband transient too. Effect is just junk clips the coach skips — cheap. | Accept some; band-check + refractory period handle most. |
| 10 | Idle bounce tripping motion-only detection | – | – | – | **84** | Motion-only detection fires on idle bouncing, not just strokes. | Motion never fires alone now — coincidence with audio (or pose-tracked swing) is required. |
| 11 | Rolling buffer evicting shots before review | – | – | – | **75** | Older shots pushed out of the buffer before the coach reviews them. | Pin shot clips; evict only non-shot video. |
| 12 | Tripod bumped, player leaves the frame region | – | – | – | **72** | Detection region no longer contains the player. | Full-frame fallback + framing overlay to re-aim. |

## Design decisions the DFMEA drove (in the build)

1. **Coincidence, not single-channel detection.** Audio hit-sound within ±150 ms of a visible swing by the tracked player. Either alone is weak; the pair is strong.
2. **Pose-gated visual as primary judge.** A lightweight pose model (MediaPipe/MoveNet class, on-device) tracks the filmed player's arm — wrist-velocity spike + elbow-arc sweep inside a ~300–500 ms window is the stroke signature. The coach taps *which person* to track at setup; other tables' smashes are invisible to it. (A local LLM is the wrong tool here: ~1–5 s per frame, non-deterministic; save the LLM for post-session clip analysis in v2, where seconds of latency are fine and language is the right output.)
3. **Manual mark as ground truth override.** Never removed; back-dated ~350 ms with 1.5 s pre-roll.
4. **Noisy-hall mode.** Demotes auto-detect to backfill; thumb becomes primary when the hall is loud.

## Field-test checklist — first live test (Green Lake, Tue 2026-10-06, 6:00–8:45 PM)

Quick checks during free play, mapped to the DFMEA rows:

- [ ] **#7 — Is the mic hearing?** Glance at the live detection meter during warm-up hits. If it stays flat, browser noise suppression is still on — restart the session with raw-mic permission.
- [ ] **#1 — Soft shots.** Hit a few deliberate soft pushes / drop shots. Do they get logged? If not, run the 5-tap per-session calibration.
- [ ] **#3 — Neighboring tables.** During a busy stretch, count how many logged "strokes" came from another table's hits. That's the coincidence gate failing — note it, it decides whether table-mic mode is needed.
- [ ] **#5 — Clip framing.** Manually mark a shot, then replay: is the contact moment centered in the clip? Late taps should be auto-corrected (~350 ms backdate + 1.5 s pre-roll).
- [ ] **#6 — Phone health.** After ~30 minutes of capture: is the phone hot? Is the page still alive? Note battery drain.
- [ ] **#4 — Consent/framing.** Keep the frame on your own table; a word to the venue covers the rest. Everything is on-device — nothing uploads.

## Beyond DFMEA — complementary frameworks

- **FMECA:** DFMEA + criticality. Adds consequence ranking for the highest-RPN items (#1–#3) so verification effort goes where it matters most.
- **FTA (fault tree analysis):** Top-down from one feared event — e.g. "tool teaches wrong form" — to find single-point failure paths #3's fixes don't cover.
- **STPA:** For the full system including the coach-in-the-loop: asks what unsafe *control actions* are possible (e.g. coach replaying the wrong player's clip as instruction).
- **DVP&R:** The natural next document — turns this checklist into a formal test matrix with pass/fail criteria and results.
- **8D:** If a field issue shows up tonight, 8D is the corrective-action format to close it.

---
*Source: DFMEA delivered in the design discussion 2026-10-06; failure modes, ratings, and headline are verbatim from that session. Field-test section added for the 2026-10-06 Green Lake live test.*
