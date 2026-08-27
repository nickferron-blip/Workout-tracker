# Workout Tracker — Handoff

Everything a fresh session needs to understand, run, and extend this app.

## What it is
A mobile/iPad-first **Progressive Web App** for logging a fixed 5-day training
split (top-set / back-off, train-to-failure methodology). Single self-contained
`index.html`, **hosted free on GitHub Pages**, syncing data through the user's
**Google Drive** (Google sign-in). Same architecture as the user's Finance
Tracker — see that app's HANDOFF for the shared pattern.

- **Live:** https://nickferron-blip.github.io/Workout-tracker/
- **Repo:** https://github.com/nickferron-blip/Workout-tracker (branch `main`)
- **Local path:** `/Users/nicolasferron/Library/CloudStorage/GoogleDrive-nick.l.ferron@gmail.com/My Drive/AI Folder/Workout-tracker`

## Architecture / persistence
- No backend, no build step. `index.html` holds all HTML/CSS/JS. Charts are
  drawn on a `<canvas>` by hand (`drawLineChart`) so there are **no CDN deps** —
  works offline at the gym. The only external script is Google Identity Services
  (`accounts.google.com/gsi/client`).
- **Data = one JSON file, `workout-data.json`, in the user's Google Drive.**
  Loaded/saved via the Drive REST API with a GIS token client; cached in
  **IndexedDB** (`workout-tracker` db, key `state`) for instant/offline paint.
  Flow: `loadState()` paints from cache → silent re-auth → pulls from Drive.
  `save()` = `_saveLocal()` (immediate) + debounced `_saveToDrive()` (PATCH).
- Key funcs: `signIn` / `signOut` / `loadState`, `_findVisibleFile` /
  `_readDriveFile` / `_createVisibleFile` (multipart) / `_saveToDrive`.
- **PWA:** `manifest.json` + `sw.js` (service worker caches same-origin shell
  only; Drive/GIS requests pass through) + `icons/` (192/512 PNG, from `icon.svg`).

## Google Cloud (its own dedicated project — separate from finance)
- Web OAuth **Client ID** (in `index.html`, `const CLIENT_ID`):
  `491987759955-krrij8805jdthf928dnqqkkr3dgirs0t.apps.googleusercontent.com`
  (an OAuth Web Client ID is public by design — safe in the client-side repo.)
- **Scope:** `https://www.googleapis.com/auth/drive.file` — per-file access, so
  the app only ever touches files **it** created (`workout-data.json`), not the
  rest of Drive. Non-sensitive scope → no Google verification needed. (Finance
  uses full `drive` because it shares the file with a partner; this app doesn't.)
- Console config: Google Drive API enabled; OAuth consent = **External /
  Testing** with the user added as a **Test user**; OAuth client's
  **Authorized JavaScript origins** = `https://nickferron-blip.github.io`
  (origin only — every Pages repo on that account shares this one origin).

## Deploy / how to ship a change
This Mac has **only `git`** (no gh/gcloud/node/brew). It still works because:
- `git push` authenticates via the **cached osxkeychain credential** (the user
  pushes from this machine, so github.com creds are in the keychain).
- GitHub Pages was enabled via the REST API using the token from
  `git credential fill` — `POST /repos/{owner}/{repo}/pages` with
  `{"source":{"branch":"main","path":"/"}}`. Never print/store that token.

To ship: edit `index.html` → `git push`. Pages redeploys in ~1–2 min. **Bump the
`CACHE` version in `sw.js`** (currently `workout-tracker-v6`) whenever you change
`index.html`, or returning clients get the old cached shell. User hard-refreshes
(or reopens the PWA). **Drive data is never touched by app updates.**

Privacy: `.gitignore` excludes `workout-data*.json` — never commit personal data.

## Model — user-editable workouts (v2)
The app is data-driven. The original hardcoded `PROGRAM` (from the user's "Rules
of Execution" doc: Chest&Biceps / Legs / Shoulders&Triceps / Back&Traps, trained
Mon/Tue/Thu/Fri) is now **only a seed source** used by `seedWorkouts()` +
`_migrateV2()` on first load. Everything the user sees is editable data:
- **Workouts** they build (name, colour, muscle groups, ordered exercises).
- **Schedule** they assign (weekday → workoutId | null) in the Program tab.
- **Equipment** they own (Settings) — filters the exercise picker.

`EXERCISE_LIB` (~95 exercises) each `{ n, m:[muscle], e:[equip], t:'c'|'i' }`.
Equipment codes: `db bb sm sq ca` (user-toggleable) · `ma` fixed/plate machine
(needs `gym`) · `bw` bodyweight (always available). `exAvailable()` gates the
picker; `gym` = everything. Adding an exercise gives it a `defaultScheme(type)`
(compound → 1 feeder + top 6–8 + back-off 8–10; isolation → 1 feeder + top 10–12).

Progression key = **slug of the exercise name** (`exKeyOf`), so it's continuous
across workouts/edits. `_migrateV2()` re-keys old `d1e1`+`variant` history entries
to name slugs and stamps each session with `workoutId/workoutName/color`.

## Data model (`state`)
```
state = {
  active: session | null,
  history: [ session ],
  workouts: [ { id, name, color, muscles:[key], mobility:[str],
                exercises:[ { id, name, muscles:[], equip:[], type:'c'|'i',
                              feeders:int, sets:[{k:'top'|'backoff', r}], note } ] } ],
  schedule: { 0..6 : workoutId | null },        // JS getDay()
  settings: { unit, restSeconds, startDate, deload, equipment:['gym'|'db'|...] }
}
session = {
  id, date, workoutId, workoutName, color, startedAt, finishedAt, deload,
  mobility:[str], mobilityDone:[i],
  entries: { [exKeyOf(name)]: { name, note, sets:[ {k, r, weight, reps, done, added?} ] } }
}
```
- Tabs: **Today** (uses `scheduledWorkoutId()`; session view when `active`),
  **Program** (schedule editor + rules + workout builder + exercise picker),
  **Progress**, **History**. Settings via header gear (bottom sheet).
- Feeder sets are seeded from `ex.feeders`; during a session `addSet(key,'feeder'|'top')`
  inserts in `KIND_ORDER` (feeder→top→back-off) and only `added:true` sets get a ✕.
- Features: rest timer (ring + beep + vibrate, auto-starts on set-check),
  "last time" overload hint + PR badge, e1RM (Epley `w*(1+reps/30)`) trend chart,
  week counter + deload nudge (weeks 5–6) + deload mode (−25%), JSON export/import.

## Status / open items
- **Done & live.** Editable workouts/schedule/equipment shipped and verified.
- Possible future work (none requested): per-exercise rep-scheme editing in the
  builder, weekly volume/tonnage summary, bodyweight log, plate-math helper,
  editing a past workout from History, custom exercises not in the library.
