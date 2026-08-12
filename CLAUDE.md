# Growth of Garden / สวนของเรา

Bilingual (Thai-default / English-toggle) animated wellbeing check-in web app for Thai medical students, deployed nationwide. Built and maintained as **ONE HTML file** (`index.html`) — all CSS, JS, copy and small SVG art embedded. No build step, no dependencies, no backend server. Heavy binaries (music, painted backgrounds) are **not** embedded; they load from a CDN — see Assets below.

Owner: Pratiparn (works from iPad only). Creative partner: Bam (approves all copy/art). A professor oversees the data collection side (student council / SMO receives the data).

## Framing rule (important)
This is a "Growth Reflection" tool, **NOT a mental-health assessment**. Never use clinical/diagnostic language in user-facing copy. The app must feel like a storybook, not a screening form. Support resources (hotline 1323, faculty counseling) appear only on Rainy/Autumn results.

## Architecture
- Stage machine: `intro → consent → story → demo (demographics, only if consented) → walk → result`
- `story[]` = tap-through intro pages (2 tip pages + 6 narrative pages). Music starts at `storyStep===2` (must remain right after the sound-on tip page).
- `WALK[]` = ordered narration/question pages. `syncFromWalk()` routes; `renderWalk()`/`renderQuestion()` display.
- i18n: everything lives in `T = {th:{...}, en:{...}}`. Both languages must ALWAYS be updated together. `toggleLang()` re-renders the current stage — every new screen needs a branch there (this has caused bugs twice).
- Scoring: 4 variables 0–1 (sunlight=motivation, water=rest, roots=support, storms=calm). `band()` maps energy×purpose to 4 gardens: blooming / growing / rainy / autumn. Options run A(best)→D(worst); value = `1 - index/3`. **Do not change thresholds without explicit approval.**
- QA override: `?band=blooming|growing|rainy|autumn` URL param forces a result screen.

## Assets (CDN, not embedded)
- `index.html` is ~71KB. It was 5.2MB until the MP3 (5.26MB as base64) and the background JPEG were pulled out — base64 also cost a 33% size penalty on top.
- Binaries live in `/assets` in the GitHub repo (`HLYJezz/Growth-of-garden`, **must stay public**) and are served by jsDelivr via the `CDN` constant in `index.html`. Current files: `music.mp3`, `bg-main.jpg`.
- To swap art: upload to `/assets` on GitHub **keeping the same filename** and it goes live. jsDelivr caches `@main` for ~12h; to publish instantly, load the same path once on `purge.jsdelivr.net/gh/...`. A *new* filename must also be entered in `ASSETS.backgrounds`.
- `ASSETS.backgrounds` slots accept four value types: a bare CDN filename, a full `data:`/`http`/`url(...)` value, or a CSS gradient string. `setBg()` handles all four.
- Every background paints `BG_FALLBACK` (a gradient) immediately, then upgrades itself once the real image loads. A failed download keeps the gradient — a blocked CDN degrades to a plain garden, never a broken page. `preloadBackgrounds()` warms all of them during the intro; `_bgState` ensures each file is fetched once.
- Music streams instead of being embedded, so it can't start instantly. `preloadMusic()` builds the element and buffers it at `goStory()`; `initAudio()` only *plays*, still at `storyStep===2`. If the track can't be fetched the mute button hides itself.
- Absolute CDN URLs (not relative paths) are deliberate: `index.html` stays portable and works from any host, a preview tool, or emailed to Bam.

## Design language (locked decisions)
- Questions are **pure garden metaphor** — options must never contain real-world words that reveal which answer is "good" (validity/social-desirability decision). The real-life mapping lives ONLY in the narration before each question (e.g., "the sunlight is what wakes you each day").
- Story premise: "this garden is planted with your days; every corner reflects your life."
- Result copy (summaries, cute quotes, key messages) is Bam-approved — don't rewrite without being asked.
- Art direction locked: naïve folk-art gouache storybook style (reference: soft blue/green/cream/terracotta, visible brushstrokes, no people, portrait 9:16). Constant prompt string exists; per-scene art is PENDING generation. Backgrounds bg0–bg6 currently all share one placeholder image intentionally.
- Planned background system (not yet built): ~9 images — gate (story, zoom-in per page), sky (bright Q1 / grey-tinted Q4), soil, roots, flowers, path, + 4 seasonal result gardens. Zoom-within-scene and tint-for-mood to stretch images across pages.

## Data collection (working, live)
- Consent screen (before story). Decline = full app works, no data sent, demographics skipped. This must never change.
- Demographics (if consented): age range (18-19/20-22/23-25/26+), year 1–6, gender (Male/Female/Prefer not to say). Stored as **indices** (language-independent); canonical English labels in `DEMO_CANON` are what get sent. Never store the display label as state (caused a bug).
- On result: one `fetch()` POST (no-cors) to Google Apps Script → Google Sheet. Payload: token, age, year, gender, 4 labeled scores ("72 (สูง)"), garden full name, language.
- Guards: `hasSentResult` (once per walk), sessionStorage session guard (**must stay wrapped in try/catch** — sessionStorage throws in sandboxed iframes; this crashed the app once), `SUBMIT_TOKEN` spam filter (honestly documented as NOT real security — endpoint is public by design, accepted tradeoff).
- Server side: Apps Script (`Code.txt` in repo/outputs) validates token + required fields, appends row. Redeploy ("New version") required after any script change — the URL stays the same.

## Working preferences (follow these)
- Batch related edits into one pass; don't drip tiny edits.
- Flag decisions/concepts for approval BEFORE building; owner reads drafts first, then says build.
- Thai copy first for review; English mirrors written after approval.
- Owner tests on iPad Safari via hosted URL (local file:// renders as text on iPadOS; sandboxed iframe previews like codeshack break sessionStorage — don't debug in those).
- After changes, verify JS with `node --check` on the extracted <script> block. (Node is NOT installed on the Windows machine — the fallback is to load the file in a browser and confirm the script block executes with no console errors, which also exercises the real runtime.)
- `index.html` is UTF-8 **without BOM**, LF line endings, ~6,379 Thai codepoints. Windows PowerShell 5.1 reads files as ANSI by default and will mangle the Thai — always force UTF-8 (`[System.IO.File]::ReadAllBytes` + a strict `UTF8Encoding`), and keep .ps1 helper scripts pure-ASCII or PS 5.1 mis-parses them too.
- The MP3 is 3.9MB at 112kbps — already compressed; don't recompress further without asking.
- When testing the walk end-to-end, **stub `window.fetch` first** or the result screen posts junk rows to the live Google Sheet.

## Known pending threads
1. Gouache art generation (owner + Bam) → then build the background zoom/tint system. The loader is already in place: upload each image to `/assets` and point the `ASSETS.backgrounds` slots at the filenames.
2. Hosting: Netlify drag-and-drop or GitHub Pages recommended; not yet set up. Because asset URLs are absolute, `index.html` can be dropped anywhere without carrying a folder alongside it.
3. An old "music not playing" report was likely device mute / preview-tool artifact; music logic is verified correct.
