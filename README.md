# Workout Tracker

A single-file, mobile/iPad-first web app for logging a 5-day training split
(top-set / back-off methodology) with a rest timer, progressive-overload hints,
strength-progress charts, and deload-week tracking.

**Live:** _enable GitHub Pages on this repo (Settings → Pages → Deploy from `main` / root)._

## How it works
- Everything is in `index.html` — no build step, no dependencies.
- Your training log is saved to a JSON file on your computer via the browser's
  **File System Access API** (desktop Chrome/Edge), which syncs through OneDrive —
  the same mechanism as the finance tracker. On iPhone/iPad the data is stored in
  the browser; use **Settings → Export / Import** to back up or move it between devices.
- Add to Home Screen on iPad/iPhone for a full-screen app.

## Editing
Just edit `index.html` and push — GitHub Pages redeploys automatically.
