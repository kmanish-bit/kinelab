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
2. Seven exercises are defined in the `EXERCISES` config object: squat
   depth, shoulder reach, trunk rotation, neck turn, single-leg balance,
   forward reach/hinge, and position & readiness.
3. Each exercise has a `frameMetric()` function that computes a simple
   joint-angle or distance-ratio value per frame (pure geometry, no ML
   training involved), and an `aggregate` mode — `min`/`max` for peak
   range-of-motion movements, `range` for the balance hold (smaller
   drift = steadier), and `avg` for the static readiness check.
4. Recording length is chosen per exercise (10 / 20 / 30s pills on the
   instructions screen), defaulting to whatever suits that movement —
   quick reps like the squat default shorter, the balance hold defaults
   longer. The aggregate value across the clip maps to one of four score
   bands per exercise, each with a plain-language note.
5. Every exercise has an original, hand-drawn SVG illustration (in the
   `ICONS` object), shown on both the exercise card and the instructions
   screen, so people aren't left parsing the movement from text alone.
   These are simple line-art stick figures, not stock photos or video —
   deliberately, to keep the file dependency-free and side-step any
   licensing question around exercise photography.
6. Session history is kept in memory only (resets on page reload) —
   intentionally, to keep this file dependency-free. Wire up Firebase
   (or any backend) once you're past validation.

## Running it

This needs a **local server or HTTPS host** — browsers block camera
access on `file://` pages and on plain HTTP.

Easiest local option:
```
cd path/to/folder
npx serve .
```
Then open the printed `localhost` URL on your phone or laptop (both work;
phone is the real target device).

To test with real users without building an app: deploy this single file
to any static host (Vercel, Netlify, GitHub Pages all work with zero
config) and share the link. HTTPS is provided automatically by all three.

## Known limitations (intentional, for this phase)

- **2D camera, so no true depth/rotation measurement.** Trunk rotation,
  shoulder reach, and the forward-lean measure in "position & readiness"
  are approximated using foreshortening and angle proxies, not real 3D
  joint angles. This is fine for a v1 "does the concept resonate" test —
  not fine for a claim of clinical accuracy.
- **Balance and readiness scores are proxy measures**, not force-plate
  or gold-standard postural assessments — they track how much a single
  landmark (hip midpoint, or ear-to-shoulder offset) drifts or deviates
  on camera, which is a reasonable relative signal but not a diagnostic
  one.
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
