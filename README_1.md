# Mobility Check — Phase 1.5

A single-file, camera-based mobility scoring app. Phase 1 validated the
core loop (record → score → one thing to work on); Phase 1.5 adds an
overall **Mobility Score**, trend tracking, persistence, optional
Firebase sync, and a weekly reminder.

## What's inside

- `index.html` — the entire app. Camera capture, pose detection, per-check
  scoring, the weighted overall score, the trend chart, local persistence,
  optional Firebase sync, and reminders all live in this one file.

## How the seven checks work (unchanged from Phase 1)

1. Uses **MediaPipe Pose (Tasks Vision)**, loaded from a CDN **on demand**
   (only when you open the camera), to detect 33 body landmarks per frame
   directly in the browser — no video is ever uploaded anywhere. The
   import is dynamic rather than a static top-level import, so a blocked
   or unreachable CDN only disables recording — the rest of the app
   (Mobility Score, history, charts) still works.
2. Seven exercises are defined in the `EXERCISES` config object: squat
   depth, shoulder reach, trunk rotation, neck turn, single-leg balance,
   forward reach/hinge, and position & readiness.
3. Each exercise has a `frameMetric()` function (pure geometry — joint
   angle or distance ratio, no ML training involved), an `aggregate` mode
   across the clip, and four score bands with plain-language notes.
4. Recording length is chosen per exercise (10/20/30s pills), defaulting
   to whatever suits that movement.

## Mobility Score (new)

Once you've completed **at least 4 of the 7 checks**, a weighted overall
score (0–100) unlocks on the **Score** tab.

- **Best-of-repeats.** If you redo a check, only your *best* score for
  that check counts toward the overall score — worse repeats never pull
  your score down. (`getBestScores()` in index.html.)
- **Weighted, and the weights rebalance around what you've actually
  done.** Each check has a base weight reflecting how much it drives
  everyday mobility:

  | Check | Weight |
  |---|---|
  | Squat depth | 18% |
  | Forward reach / hinge | 16% |
  | Single-leg balance | 16% |
  | Shoulder reach | 14% |
  | Trunk rotation | 14% |
  | Position & readiness | 12% |
  | Neck turn | 10% |

  If you've only done, say, 5 of the 7, those 5 weights are renormalized
  to sum to 100% — so your score is never artificially dragged down just
  for having fewer data points, but doing more checks always sharpens its
  accuracy. (`computeMobilityScore()`.)
- **Verdict bands:** 0–49 *Need to Work*, 50–69 *Doing Good*, 70–84
  *Very Good*, 85–100 *Excellent*. (`VERDICT_BANDS`.)
- **Recommendation.** The app calls out your single weakest completed
  check (reusing that check's own band note, e.g. "Tight hips or ankles
  from sitting are the usual cause…") and, if any checks are still
  untried, names them and explains that completing them sharpens the
  score. (`getRecommendation()`.)
- Weights are a starting point, not a clinical standard — see *Known
  limitations* below and tune `SCORE_WEIGHTS` once you have real usage
  data or a physio's input.

## Five movement-quality dimensions (new): Move, Align, Stability, Control, Recovery

Alongside the single overall score, the Score tab also rolls the 7 checks
up into 5 coarser dimensions — so "what should I focus on" has an answer
at both the check level and the movement-quality level.

| Dimension | What it means | Fed by |
|---|---|---|
| **Move** | Range of motion — how far you can move through your hips, shoulders and neck. | Squat depth, Shoulder reach, Neck turn |
| **Align** | How well your body is positioned and stacked, at rest and in motion. | Position & readiness |
| **Stability** | How steady you stay under an unstable, single-limb position. | Single-leg balance |
| **Control** | How controlled and symmetric your movement is through a range — not just how far it goes. | Trunk rotation, Forward reach/hinge |
| **Recovery** | A proxy for resting readiness. | Position & readiness |

The mapping isn't an arbitrary new taxonomy — it follows each check's own
`category` copy already in `EXERCISES` (squat's says "range", trunk's and
hinge's say "control", balance's says "stability", readiness's says
"position & recovery"). `DIMENSIONS` and `EXERCISE_TO_DIMENSIONS` near
the Mobility Score model in index.html define and reverse-map it, and
each exercise card / instructions screen shows a small tag (e.g. "MOVE",
"ALIGN · RECOVERY") naming which dimension(s) it feeds.

- **Same scoring mechanics as the overall score, just scoped to a
  dimension's checks:** best-of-repeats, and weights renormalize across
  whichever of that dimension's checks you've actually done
  (`computeDimensionScores()`). A dimension shows "—" with a "Try: …"
  prompt until at least one of its checks is completed.
- **Recovery has no dedicated check today.** There's no movement in the
  current 7 that's really *about* recovery (that's more naturally rest-
  day balance, sleep, or self-reported soreness than a camera check).
  Position & readiness is the closest available proxy — its own copy
  already names "recovery" — so it feeds both Align and Recovery until a
  dedicated recovery signal exists. This is called out directly in the
  UI's dimension blurb, not hidden.
- **A second recommendation, at the dimension level.** The Score tab
  shows a "Focus dimension" card naming your single weakest *unlocked*
  dimension, the specific check driving it down, and that check's own
  improvement note — plus which dimensions still need more data to
  unlock. (`getDimensionRecommendation()`.) This sits alongside, not
  instead of, the existing per-check "Focus area" recommendation.

## Trend: daily, weekly, monthly (new)

The Score tab includes a line chart with Day / Week / Month toggles.

- **Daily** point = the Mobility Score *as of that day*, computed from
  your cumulative best-scores-to-date (so it reflects your progress over
  time, not just that day's session). One point per day that had enough
  cumulative checks to compute a score.
- **Weekly / Monthly** points = the average of the daily points falling
  in that ISO week / calendar month.
- The chart needs at least two points in the selected range to draw a
  line; otherwise it shows a short prompt to keep checking in.
- Built as a small inline SVG line chart (hover for a hoverable
  crosshair + tooltip), no charting library — kept the file
  dependency-free. Styled with the dataviz skill's checks in mind: a
  single series needs no legend, thin 2.5px line, recessive gridlines,
  rounded data-ends.

## Persistence (new)

Every session (and a `lastReminderNotifiedTs` marker) is saved to
`localStorage` under the key `kinelab_mobility_v1` immediately after each
check — history survives reloads even with no backend configured. This
is the fallback that always works; Firebase (below) is optional and
layers cross-device sync on top of it, never replaces it.

## Firebase — Auth + Firestore (new, optional)

**Status: already configured and live.** This app is wired up to the
`kinelab-mobility-check` Firebase project — Anonymous and Google sign-in
are enabled, Firestore (Standard edition, production mode) is created,
the security rules below are published, and the real config is already
pasted into `index.html`. Anonymous sign-in kicks in automatically the
first time the app loads with a network connection; nothing further is
needed to start syncing.

One thing to do before deploying to a real domain (Vercel/Netlify/GitHub
Pages, etc.): Google's sign-in popup only works from domains Firebase
recognizes. `localhost` and `kinelab-mobility-check.firebaseapp.com` /
`.web.app` are authorized by default — add your deployed domain in
Firebase console → Authentication → Settings → Authorized domains before
"Sign in with Google" will work there (Anonymous sign-in and Firestore
reads/writes aren't affected by this list, only the Google popup is).

The steps below are kept for reference — e.g. if you ever need to point
this app at a different Firebase project, or set one up for a fork:

**To enable it:**

1. Create a Firebase project (or use an existing one) at
   [console.firebase.google.com](https://console.firebase.google.com).
2. Enable **Authentication** → Sign-in method → turn on *Anonymous* and
   *Google*.
3. Enable **Firestore Database** (production mode is fine — rules below
   lock it down).
4. Project settings → General → "Your apps" → add a Web app → copy the
   config object.
5. Paste it into the `firebaseConfig` object near the bottom of
   `index.html` (search for `YOUR_API_KEY`).
6. Add these Firestore security rules so users can only read/write their
   own history:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/sessions/{sessionId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```

**Behavior once configured:**

- On load, the app signs the user in **anonymously** automatically (no
  UI needed) so history starts syncing right away.
- A "Sign in with Google" link appears on the Score tab so a guest can
  upgrade to a persistent identity and keep history across devices/
  browsers.
- Sessions are written to `users/{uid}/sessions/{sessionId}` in
  Firestore as they're recorded, and merged with local history on
  sign-in (local-only sessions get uploaded; remote sessions get pulled
  down) — so nothing is lost switching between local-only and signed-in
  use.
- Streaks and the Mobility Score are computed the same way whether the
  data came from Firestore or localStorage, since Firestore is just kept
  in sync with the same `sessionHistory` array the rest of the app reads.
- If Firebase init fails for any reason (bad config, offline, CDN
  blocked), the app logs a warning and continues in local-only mode —
  it never blocks the core experience.

## Weekly reminder (new)

- The app tracks the timestamp of your most recent session. If **7+
  days** have passed, a dismissible banner appears on the Checks tab.
- An "Enable reminders" button requests browser Notification permission;
  once granted, the app also fires a native browser Notification (at
  most once per 7-day window, re-checked hourly while the app is open).
- This is intentionally simple and client-side — no push server/VAPID
  keys. It only fires while the app/tab is open (or backgrounded but
  still running), which fits a lightweight Phase 1.5 reminder loop. A
  true background push (via a Service Worker + push server) is a natural
  next step once this validates that reminders bring people back.

## Streaks

A simple "N-day streak" chip on the Score tab counts consecutive days
(ending today or yesterday) with at least one completed check —
computed locally from session history, so it works identically with or
without Firebase.

## Running it

This needs a **local server or HTTPS host** — browsers block camera
access on `file://` pages and on plain HTTP.

```
cd path/to/folder
npx serve .
```
Then open the printed `localhost` URL on your phone or laptop.

To test with real users: deploy this single file to any static host
(Vercel, Netlify, GitHub Pages) and share the link. HTTPS is automatic
on all three, and if you've configured Firebase, add that host's domain
to Firebase Auth's "Authorized domains" list too.

## Known limitations (intentional, for this phase)

- **2D camera, so no true depth/rotation measurement** — unchanged from
  Phase 1; trunk rotation, shoulder reach, and forward-lean are
  foreshortening/angle proxies, not real 3D joint angles.
- **Score thresholds and Mobility Score weights are placeholder
  estimates**, not derived from a physiotherapist-reviewed reference
  range. Treat both as a starting point to refine with real usage data.
- **Recovery dimension has no dedicated check** — it's currently proxied
  entirely by the Position & readiness check (see *Five movement-quality
  dimensions* above). Treat it as directional, not a real recovery
  measure, until a dedicated signal is added.
- **Reminders only fire while the app/tab is open** — no server-side
  push yet (see above).
- **Firestore sync is "good enough for a prototype," not conflict-safe**
  for true concurrent multi-device writes — session IDs are
  `{exerciseKey}-{timestamp}`, so the same device recording twice within
  the same millisecond (not realistic) would collide; multi-device
  simultaneous writes to *different* checks merge fine.
- **Single-person detection only** (`numPoses: 1`).
- **Language is deliberately non-clinical** — stays in wellness/fitness
  territory, not diagnostic.

## Natural next steps

1. Replace placeholder score bands/weights with physio-reviewed
   reference ranges once real usage data exists.
2. Give Recovery its own dedicated signal (rest-day balance, self-
   reported soreness/sleep, or a wearable integration) instead of
   proxying it off Position & readiness.
3. Server-side push notifications (Service Worker + VAPID) so weekly
   reminders reach people even when the app isn't open.
4. Real-time in-exercise feedback (harder), more exercises, or tying
   scores to the apparel brand's community features.
