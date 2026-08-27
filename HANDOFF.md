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
`CACHE` version in `sw.js`** (currently `workout-tracker-v3`) whenever you change
`index.html`, or returning clients get the old cached shell. User hard-refreshes
(or reopens the PWA). **Drive data is never touched by app updates.**

Privacy: `.gitignore` excludes `workout-data*.json` — never commit personal data.

## The program (source of truth = the user's "Rules of Execution" doc)
Schedule: Mon=Day1, Tue=Day2, Wed=rest, Thu=Day4, Fri=Day5, Sat/Sun=rest
(`PROGRAM.schedule`, keyed by JS `getDay()`).
- **Day 1 — Chest & Biceps** (incline press, flat press/dips, flyes/crossovers, EZ/DB curl, incline/preacher curl)
- **Day 2 — Legs** (ham curl, hack/leg press, leg ext, RDL/hyperext, calf raise)
- **Day 4 — Shoulders & Triceps** (OHP, lateral raise ×2, rear delt ×2, pushdown, overhead ext)
- **Day 5 — Back & Traps** (chest-supported row, pulldown, single-arm row, shrugs ×2)

Each exercise: `{ key, variants[], feeders, note?, sub?, sets:[{k:'top'|'backoff', r:'6–8'}] }`.
`key` (e.g. `d1e1`) is stable across variant choice → used for history &
progression. Rules of execution + per-day mobility are shown on the Program tab.

## Data model (`state`)
```
state = {
  active: session | null,        // in-progress workout
  history: [ session, ... ],     // completed
  settings: { unit:'lb'|'kg', restSeconds:165, startDate:'YYYY-MM-DD', deload:bool }
}
session = {
  id, date:'YYYY-MM-DD', dayId, startedAt, finishedAt, deload:bool,
  entries: { [exKey]: { variant:int, sets:[ {k, r, weight, reps, done} ] } },
  mobilityDone: [ indices ]
}
```
- Tabs: **Today** (auto-picks scheduled day; shows session view when `active`),
  **Program**, **Progress**, **History**. Settings via header gear (bottom sheet).
- Features: rest timer (ring + beep + vibrate, auto-starts when a set is checked),
  "last time" progressive-overload hint + PR badge, estimated 1RM (Epley:
  `w*(1+reps/30)`) trend chart per exercise, week counter + deload nudge
  (weeks 5–6) + deload mode (−25% targets), mobility checklist, JSON export/import.

## Status / open items
- **Done & live.** User has signed in and it syncs. No outstanding bugs known.
- Possible future work (none requested yet): weekly volume/tonnage summary,
  bodyweight log, plate-math helper, auto-detect deload from week counter,
  per-set RPE/notes, editing a past workout from History.
