# Board — setup guide

Three pieces: the app (GitHub Pages), the sync layer (your existing PocketBase on Railway),
and a small photo-reading proxy (new Railway service). None of it needs your GitHub or
Railway login on my end — you deploy it, these steps just walk through it.

## 1. PocketBase — add the collection

In your PocketBase admin UI (the same instance DAYSCORE uses):

1. Create a new collection called `board_tasks`.
2. Add fields:
   - `text` — Plain text, required
   - `done` — Bool, default false
   - `completedAt` — Date, not required
3. Under **API Rules**, set List/View/Create/Update/Delete rules to blank (public) for now —
   same trust model as your other personal tools. Tighten later with auth if you want.

## 2. Photo-reading backend — new Railway service

1. Push the `board-backend` folder to its own GitHub repo (or a subfolder Railway can target).
2. In Railway: New Service → Deploy from GitHub → point at that repo.
3. Set one environment variable: `ANTHROPIC_API_KEY` — get this from
   https://console.anthropic.com/settings/keys
4. Railway will give you a public URL like `https://board-backend-production.up.railway.app`.
   Test it by visiting the URL — you should see "Board extraction backend is running."

## 3. The app — GitHub Pages

1. Create a repo, e.g. `18ventures/board`.
2. Push everything in this `board-app` folder (index.html, manifest.json, sw.js, icons/) to it.
3. In `index.html`, fill in the two config lines near the top of the `<script>` block:
   ```js
   const POCKETBASE_URL = "https://your-actual-pocketbase.up.railway.app";
   const EXTRACT_BACKEND_URL = "https://your-actual-board-backend.up.railway.app";
   ```
4. In repo Settings → Pages, set source to the branch/folder you pushed to.
5. Your app will be live at `https://18ventures.github.io/board/`.

## 4. Add to your phone's home screen

- **iPhone (Safari):** open the URL → Share icon → "Add to Home Screen."
- **Android (Chrome):** open the URL → ⋮ menu → "Add to Home screen" / "Install app."

It'll open full-screen, no browser chrome, just like DAYSCORE.

## Notes

- Tasks sync live across devices through PocketBase's realtime subscriptions — no refresh needed.
- If PocketBase or the backend is unreachable, the status dot at the top turns grey and tells you.
- The service worker only caches the app shell (HTML/manifest/icons) — task data always goes
  live to PocketBase, so you never see stale tasks.
- Locking down CORS on the backend to your GitHub Pages origin (rather than `*`) is a good
  next step once it's all working — the commented-out line in `server.js` shows where.
