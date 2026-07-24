# Spanish Quiz Tracker

A single-page React app (no build step required) for tracking Spanish lessons, exams, and routine drills — due dates, scores, pass/fail grading, a weekly/monthly schedule view, a completion history log, and progress charts.

Everything runs client-side. Data is stored in the browser's `localStorage`; nothing is sent to a server.

## Features

- Table, This Week, This Month, Charts, and History views
- Sortable table columns (click a header, or use the "Sort by" control on mobile) with ascending/descending toggle
- Progress charts: score trend over time, average score by subject, completion by subject
- Custom pass thresholds per subject
- Routine (recurring weekly) vs. one-off quizzes, with automatic weekly reset + archiving
- JSON export/import for backups
- Optional: sync one shared quiz library to Google Drive so multiple people (e.g. a teacher and students) can see the same data — see **Setup → Google Drive sync** below

## Setup

### Google Drive sync (optional)

By default the app only uses `localStorage` — nothing leaves the browser. If you want several people to share one quiz library through Drive instead:

1. In [Google Cloud Console](https://console.cloud.google.com), enable the **Google Drive API** for a project (APIs & Services → Library).
2. APIs & Services → Credentials → **Create Credentials → OAuth client ID** → Application type **Web application**.
3. Under **Authorized JavaScript origins**, add every exact origin this app will be served from (e.g. `http://localhost:8000` and `https://<your-username>.github.io`). No redirect URI is needed.
4. Open `index.html`, find `const DRIVE_CLIENT_ID = ""` near the top of the script, and paste your Client ID in.
5. Find `const ADMIN_EMAILS = []` just below it and add the Google account email(s) that should be allowed to **edit** the shared data (e.g. `["teacher@gmail.com"]`). Anyone else who connects gets **read-only** access in the app. Leave this empty only if you're fine with every connected account having full edit access — the app will show a warning banner if so.
6. Open the app, use **⋯ → Connect Google Drive**, and sign in. The first admin to connect creates the shared file in their own Drive; share that file with anyone else who needs access (via Drive's normal sharing UI), and they can connect and read it too.
7. Optional but recommended: after that first connect, check the browser console for a line like `Drive file created. Paste this into DRIVE_SHARED_FILE_ID: <id>`. Paste that ID into `const DRIVE_SHARED_FILE_ID = ""` in `index.html`. This makes every future connection look up that exact file by ID instead of searching Drive by name — deterministic, and avoids ambiguity if a second file with the same name ever exists. Leaving it blank still works via name search, just less precisely.

Notes:
- This app uses the `drive.file` scope to create/edit the file (so it never gets broad write access to your whole Drive) plus `drive.readonly` so accounts that didn't create the file can find and read it once it's shared with them — see the comments above `DRIVE_SCOPES` in `index.html` for the trade-off that entails.
- Concurrent edits are handled with a conflict check: if the shared file changed since your last sync, the app stops and asks you to choose "keep mine" or "load theirs" instead of silently overwriting anyone's work.
- The Drive access token itself is never saved to `localStorage` — only a "try to reconnect automatically" flag.

## Running it locally

No build tooling, no `npm install` — it's one static HTML file.

```bash
# from the project folder
python3 -m http.server 8000
# then open http://localhost:8000
```

Or just double-click `index.html` to open it directly in a browser (everything, including `localStorage`, still works from a `file://` URL in most browsers).

## Project structure

```
spanish-quiz-tracker/
├── index.html          # the entire app: markup, styles, and React source
├── sample-backup.json  # example backup file, importable via the app's "Import backup" menu
└── README.md
```

`index.html` loads React, ReactDOM, and Babel Standalone from a CDN (`cdnjs.cloudflare.com`) and compiles the embedded JSX in the browser at load time. There is no `package.json`, bundler, or build output — what you edit is what ships.

## Putting this on GitHub

1. **Create the repo.**
   - On GitHub: click **New repository**, name it (e.g. `spanish-quiz-tracker`), leave it empty (no README/license/gitignore — you already have those here), and create it.
2. **Push this folder.**
   ```bash
   cd spanish-quiz-tracker
   git init
   git add .
   git commit -m "Initial commit: Spanish Quiz Tracker"
   git branch -M main
   git remote add origin https://github.com/<your-username>/spanish-quiz-tracker.git
   git push -u origin main
   ```
3. **(Optional) Host it for free with GitHub Pages.**
   - Repo → **Settings** → **Pages**.
   - Under "Build and deployment", set **Source** to **Deploy from a branch**.
   - Branch: `main`, folder: `/ (root)` → **Save**.
   - After a minute or two, your app is live at `https://<your-username>.github.io/spanish-quiz-tracker/`.
   - Note: `localStorage` is per-browser and per-origin. Data you enter on the GitHub Pages URL won't automatically show up when you open `index.html` locally, and vice versa — they're different origins. Use **Export backup** / **Import backup** to move data between them.

## Backups

Use the **⋯ (more actions) → Export backup** menu item to download a timestamped JSON snapshot of all subjects, quizzes, settings, and history. **Import backup** restores from one of these files (this replaces all current data, with a confirmation prompt first). `sample-backup.json` in this repo is an example of that format you can use to try the app out.

## Browser support

Any modern evergreen browser (Chrome, Firefox, Safari, Edge). Requires JavaScript and `localStorage` to be enabled.
