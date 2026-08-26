# Deploy the Workout Tracker

Same setup as the Finance Tracker. If you host under the **same GitHub account**
as the finance app, you can reuse its Google Client ID and skip Part 2 entirely.

---

## Part 1 — Put the app online (GitHub Pages)

1. Create a repo on **github.com** (e.g. `workout-tracker`), **Public**.
2. Upload the contents of this `workout-app` folder: `index.html`, `manifest.json`,
   `sw.js`, `icon.svg`, and the `icons/` folder. (Or push with GitHub Desktop.)
3. Repo → **Settings → Pages** → Source: **Deploy from a branch**, Branch **`main`**,
   folder **`/ (root)`** → **Save**.
4. Wait ~1–2 min. Live at `https://<your-username>.github.io/workout-tracker/`.

## Part 2 — Google sign-in

**Reusing the finance app's Client ID (easiest):** because both apps live on
`https://<your-username>.github.io`, the finance OAuth client already allows this
one. Leave `CLIENT_ID` in `index.html` as-is and go to Part 3. If sign-in later
throws an origin/redirect error, open your existing OAuth client in
**console.cloud.google.com → APIs & Services → Credentials** and confirm
`https://<your-username>.github.io` is listed under **Authorized JavaScript origins**.

**Making a dedicated client (optional):** follow the finance app's
`SETUP-YOUR-OWN-COPY.md` Part 2 — new Google Cloud project, enable **Google Drive
API**, OAuth consent (External / Testing, add yourself as a test user, add the
`.../auth/drive` scope), create a **Web** OAuth Client ID with your Pages origin,
then paste it into `CLIENT_ID` in `index.html`.

## Part 3 — Open it

1. Go to your URL, tap **Sign in with Google**.
2. On the "unverified app" screen (normal for a personal app), choose
   **Advanced → continue**, then **Allow** the Drive permission.
3. Your log now saves to `workout-data.json` in your Drive and syncs everywhere.

**Install like an app:** iPhone/iPad → Safari → Share → **Add to Home Screen**.
Desktop Chrome/Edge → install icon in the address bar.
