# Board — current setup (as it actually exists)

Three pieces, all already built and live:

1. **PocketBase** — your existing instance (also used by DAYSCORE), collection `task_manager`
2. **task-manager** (Railway) — the backend: reads whiteboard photos, sends push notifications
3. **Task-app** (GitHub Pages) — the app itself, on your phone's home screen

No new repos needed. Everything below is either already done or is the next thing to check.

## PocketBase — `task_manager` collection

Fields that should exist:
- `text` — Text, required
- `done` — Bool
- `completedAt` — Date
- `order` — Number
- `duration` — Text (`7h` / `7d` / `7w` / empty for aspirations)
- `dueAt` — Date
- `subtask` — JSON (note: singular "subtask", not "subtasks")
- `isAspiration` — Bool
- `focus` — Bool

API rules: List/View/Create/Update/Delete all left blank (public) — same trust model as your other personal tools.

**Still needed for push notifications:** a second collection, `push_subscriptions`, with:
- `endpoint` — Text, required
- `subscription` — JSON, required

Same public rules as above.

## task-manager (Railway backend)

This is the small Node/Express service — repo `18ventures/task-manager`. It does two jobs:
1. Reads whiteboard photos (the `/extract-tasks` endpoint)
2. Sends scheduled push notifications (11pm daily, Sunday 6pm weekly)

Environment variables it needs set in Railway:
```
ANTHROPIC_API_KEY=sk-ant-...
POCKETBASE_URL=https://pocketbase-production-2a23.up.railway.app
VAPID_PUBLIC_KEY=BGBV7r-hNkAfCleA996g_AmtH0iEfCdwxH0b_aa5RhVhAtqWf_rRZAMqCU99Lzwq3O8GDkGXvQzuTaSwMvawHdo
VAPID_PRIVATE_KEY=H0IrJudChXRkSkv3gjSfyCHwmjD0m28UF4nKROQk0WI
```

**Check this:** open your Railway project → task-manager service → Settings → Networking, and confirm
the public URL. In `index.html`, the `EXTRACT_BACKEND_URL` constant currently still says
`https://YOUR-BOARD-BACKEND.up.railway.app` — if that's never been swapped for the real URL,
photo capture won't work yet (push notifications don't depend on this, they'll work regardless).

Test the backend directly any time: visit `https://your-real-url.up.railway.app/test-push` —
you should get a push notification within a few seconds if you're subscribed.

## Task-app (GitHub Pages)

Repo `18ventures/Task-app`. Config lines near the top of `index.html`'s `<script>` block:
```js
const POCKETBASE_URL = "https://pocketbase-production-2a23.up.railway.app";
const EXTRACT_BACKEND_URL = "..."; // see note above
const COLLECTION = "task_manager";
```

Deployed via Settings → Pages, source: root of `main`. Icons live at the repo root (not in a
subfolder), and the code points at them there.

Add to home screen: Share (iOS) or ⋮ menu (Android) → "Add to Home Screen." On iOS, push
notifications only work once it's actually installed this way, not just opened in Safari.

## Turning on notifications

Tap the bell icon in the header, allow notifications when prompted. Turns green once subscribed.
Do this once per device you want notified.

## Notes

- Tasks sync live across devices via PocketBase realtime — no refresh needed, when it's working.
- The service worker (`sw.js`) is network-first as of the last fix — it always checks GitHub for
  the newest code before falling back to a cached copy, so future updates should reach your phone
  without needing a reinstall.
- Auto-archive: completed Board tasks disappear after 7 days. Aspirations never auto-archive.
- Notification schedule (Europe/London time, regardless of where Railway's servers run):
  11pm daily, Sunday 6pm weekly. Both are just cron lines in `server.js` if you want to change them.
