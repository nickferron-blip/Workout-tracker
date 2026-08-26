# Workout Tracker

A mobile/iPad-first Progressive Web App for a 5-day training split
(top-set / back-off methodology) — rest timer, progressive-overload hints,
strength-progress charts, deload-week tracking, and post-workout mobility.

Built the same way as the Finance Tracker: a single-file app **hosted free on
GitHub Pages**, syncing data through **Google Drive** (Google sign-in).

- **Repo:** _this repo_
- **Live:** enable GitHub Pages (Settings → Pages → Deploy from `main` / root)

## How data works
- Sign in with Google → the app keeps one file, **`workout-data.json`**, in your
  Google Drive and caches it locally (IndexedDB) for instant, offline load.
- Because it's in *your* Drive, the log syncs across every device you sign in on
  (iPhone, iPad, desktop). No server, no shared database.
- **Settings → Export** is an extra manual JSON backup.

## Google sign-in (Client ID)
`index.html` has one constant near the top of the script:

```js
const CLIENT_ID = '…apps.googleusercontent.com';
```

GitHub Pages serves every repo of one account from the **same origin**
(`https://<user>.github.io`), so the Finance Tracker's existing OAuth Client ID
already authorizes this app — no new Google Cloud setup needed if you host under
that account. To host elsewhere, or if sign-in errors on origin/redirect, see
[SETUP.md](SETUP.md).

## Files
`index.html` (the app) · `manifest.json` + `sw.js` + `icons/` (PWA / offline) ·
`icon.svg` (source icon). Personal data (`workout-data.json`) is **git-ignored** —
it lives in Drive, never in the repo.

## Editing
Edit `index.html` and push — GitHub Pages redeploys in ~1–2 min. Bump the
`CACHE` version string in `sw.js` when you want clients to pick up changes
immediately. Your Drive data is untouched by app updates.
