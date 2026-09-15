# kinelab
Measure your mobility score

# Mobility Check — Phase 1 MVP

A single-file, camera-based mobility scoring prototype. No backend, no build
step, no app store — this is built purely to test one question: **will
people actually record themselves and come back for a score?**

## What's inside

- `mobility-mvp.html` — the entire app. Camera capture, pose detection,
  scoring logic, and UI all live in this one file.

## How it works

1. Uses **MediaPipe Pose (Tasks Vision)**, loaded from Google's CDN, to
   detect 33 body landmarks per frame directly in the browser — no video
   is ever uploaded anywhere.
2. Four exercises are defined in the `EXERCISES` config object: squat
   depth, shoulder reach, trunk rotation, neck turn.
3. Each exercise has a `frameMetric()` function that computes a simple
   joint-angle or distance-ratio value per frame (pure geometry, no ML
   training involved).
4. After a 10-second recording, the best (max or min, depending on the
   exercise) value across all frames is mapped to one of four score
   bands, each with a plain-language note.
5. Session history is kept in memory only (resets on page reload) —
   intentionally, to keep this file dependency-free. Wire up Firebase
   (or any backend) once you're past validation.


## Known limitations (intentional, for this phase)

- **2D camera, so no true depth/rotation measurement.** Trunk rotation
  and shoulder reach are approximated using foreshortening and angle
  proxies, not real 3D joint angles. This is fine for a v1 "does the
  concept resonate" test — not fine for a claim of clinical accuracy.
- **Score thresholds are placeholder estimates**, not derived from a
  physiotherapist-reviewed reference range. Treat the four bands per
  exercise as a starting point to refine once you have real usage data
  (and ideally a physio's input).
- **No login, no cloud storage, no cross-device history.** Everything
  resets on reload. This is deliberate — don't build persistence until
  you know people care about a "streak."
- **Single-person detection only** (`numPoses: 1`) — fine for this use
  case.
- **Language is deliberately non-clinical** ("range," "check," "note")
  rather than diagnostic terms, to stay clearly in wellness/fitness
  territory rather than implying a medical assessment.

## What to test with real users first

- Do people complete a full record → score loop without help?
- Do they come back and try a second exercise, or a second day?
- Do they share their score unprompted (validates the community hook)?
- Which exercise, if any, feels most "worth it" to them?

## Natural next steps (only after the above shows signal)

1. Replace placeholder score bands with physio-reviewed reference ranges.
2. Add Firebase (Auth + Firestore) for real history and streaks.
3. Add a simple weekly reminder / notification loop.
4. Only then consider: real-time in-exercise feedback (harder), more
   exercises, or tying scores to the apparel brand's community features.
