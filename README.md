# tecsec — web / home-screen version

A single self-contained page. No build step, no CLI, no Apple Developer
account. Deploy it to GitHub Pages and add it to your home screen.

Files: `index.html`, `manifest.json`, `icon-180.png`, `icon-192.png`,
`icon-512.png`.

---

## Deploy — iPhone only, ~5 minutes

1. Files app → long-press `tecsec-web.zip` → **Uncompress**.
2. github.com → `aA` → **Request Desktop Website**.
3. New repository, name it `tecsec-web`, and set it **Public**.
   GitHub Pages requires a public repo on free accounts. This is fine here
   because no API keys are in these files — see "Keys" below.
4. **Add file → Upload files** → select all five files → Commit.
5. Repo **Settings** → **Pages** → under "Build and deployment", set
   Source = **Deploy from a branch**, Branch = **main**, folder = **/ (root)**
   → Save.
6. Wait 1–2 minutes. The Pages section will show your URL:
   `https://<your-username>.github.io/tecsec-web/`
7. Open that URL **in Safari** (not Chrome).
8. **Share** button → **Add to Home Screen** → Add.

Launching from the home-screen icon runs it fullscreen with no Safari
address bar. It behaves like an app, sits next to your other apps, and
respects the notch and home indicator.

**It must be added from Safari.** Chrome and Firefox on iOS cannot create a
proper home-screen web app.

## Editing later

Edit `index.html` directly in GitHub's web editor (pencil icon) and commit.
Pages redeploys in about a minute. Hard-refresh in Safari to see changes.

---

## Keys

Keys are entered in the app under **Settings** and stored in `localStorage`
on your device. They are deliberately **not** in these files.

This is a real improvement over the native build, not just a workaround. In
the iOS version, `EXPO_PUBLIC_*` keys were compiled into the IPA and were
extractable by anyone who obtained the app. Here they never leave your phone
except in the API calls themselves, and nothing sensitive is ever committed
to a public repo.

Caveat: clearing Safari website data erases them, and you'll re-enter them.

---

## The one thing that may not work: CORS

Both live-data features call third-party APIs directly from the browser. That
only works if those APIs send `Access-Control-Allow-Origin` headers. I could
not verify this from my environment — my network access is restricted to an
allowlist that doesn't include either API — so treat the following as
expectations, not confirmed facts:

| API | Expectation | Confidence |
|---|---|---|
| **Polygon** (Markets) | Should work — Polygon documents browser use | Moderate |
| **FRED** (Bonds) | Likely **blocked** — FRED is widely reported not to send CORS headers | Moderate-to-high |

The app handles this explicitly rather than failing silently. If a request is
blocked you get a message saying so, distinguished from a bad-key error (HTTP
401/403) and a rate-limit error (HTTP 429), and the static figures remain
visible. So you'll be able to tell which problem you actually have.

**If FRED is blocked**, the options are:
- Accept static Treasury figures and update them by hand.
- Put a small proxy in front of FRED (a Cloudflare Worker is free and takes
  about 10 lines). This also lets the key live on the server instead of the
  device. Ask and I'll write it.
- Switch to a different rates source that permits browser calls.

---

## What carries over from the native build, and what doesn't

**Works the same:**
Feed with filter chips, Markets, Bonds, AI tabs, Space Grotesk / Manrope
typography, scrolling gold marquee, live Polygon refresh (rate-limit-aware,
sequential), live FRED refresh (CORS permitting), YouTube media cards.

**Better here:**
- Keys never ship inside the artifact.
- Media cards open YouTube in a real tab. The native version used an in-app
  webview; YouTube blocks iframing its search results, so a tab is both more
  reliable and what you'd actually want.
- Edits go live in a minute instead of requiring a full rebuild and
  TestFlight submission.

**Lost:**
- **Push notifications.** iOS 16.4+ does support web push for home-screen
  apps, but it needs a service worker plus a push server. Not included. The
  Alerts tab is gone; its explanation lives under Settings. If real alerts
  matter, the native TestFlight build is the path.
- **Offline support.** No service worker, so it needs a connection to load.
  Deliberate: a cache-first worker would make your own edits appear not to
  take effect, which is a bad trade while you're still changing things.

---

## Testing performed

`index.html` was executed in a headless DOM, not merely eyeballed:

- Loads with zero runtime errors
- All 5 tabs render and switch correctly
- All 6 filter chips return the right row counts (17 / 5 / 2 / 2 / 4 / 4)
- Media card opens the correctly URL-encoded YouTube search
- Key save writes to localStorage and flips the LIVE indicator
- FRED: CORS block, HTTP 400, and success paths all produce correct output
- Polygon: 429 rate limit, 401 bad key, CORS block, and full success paths
  all produce correct output, and partial results are retained on rate limit

One bug was found and fixed this way: CORS detection originally used
`e instanceof TypeError`, which fails across JavaScript realms. It now checks
`e.name === "TypeError"`.
