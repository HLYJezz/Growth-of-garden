# Growth of Garden / สวนของเรา

Bilingual (Thai-default / English-toggle) animated wellbeing check-in web app for Thai medical students, deployed nationwide. Built and maintained as **ONE HTML file** (`index.html`) — all CSS, JS, copy and small SVG art embedded. No build step, no dependencies, no backend server. Heavy binaries (music, painted backgrounds) are **not** embedded; they load from a CDN — see Assets below.

Owner: Pratiparn (works from iPad only). Creative partner: Bam (approves all copy/art). A professor oversees the data collection side (student council / SMO receives the data).

## Framing rule (important)
This is a "Growth Reflection" tool, **NOT a mental-health assessment**. Never use clinical/diagnostic language in user-facing copy. The app must feel like a storybook, not a screening form. Support resources (hotline 1323, faculty counseling) appear only on Rainy/Autumn results.

## Architecture
- Stage machine: `intro → consent → story → demo (demographics, only if consented) → walk → result`
- `story[]` = tap-through intro pages (2 tip pages + 6 narrative pages).
- **Music starts on the reader's first tap, anywhere** (`armMusicOnFirstTap()`, armed at `DOMContentLoaded`) — owner's decision, replacing the old `storyStep===2` trigger. Autoplay policy means this is the earliest it can legally begin; on the intro that tap is the "enter the garden" button. The `storyStep===2` call remains as a harmless idempotent fallback. Note the 🔊 sound-tip story page now appears *after* music has already started; it still earns its place for readers with the device muted.
- **Restart does not touch the music.** It used to `pause()` and null the element, so "start over" restarted the track from the top and felt like a page reload. The reader's mute choice is preserved too.
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
- Music streams instead of being embedded, so it can't start instantly. `preloadMusic()` builds the element and starts buffering at `DOMContentLoaded`; `initAudio()` only *plays*, on the first tap. If the track can't be fetched the mute button hides itself.

## Save my garden (share image)
- The save button used to `alert('บันทึกแล้ว')` and save nothing — a button that lied. It now renders a real 1080×1920 PNG.
- Drawn with **canvas primitives**, not a DOM screenshot, to keep the no-dependency rule (no html2canvas) and to get a portrait card sized for a phone story rather than a cropped page. Contents: garden name, sub, the tree, four score bars, Bam's quote, footer.
- `TREE_TINTS` is the single source of truth for the seasonal tree palette — both the inline SVG on screen and the canvas tree read from it, so the saved image matches what the student was looking at. `TREE_EXTRA` holds each season's canopy detail; `ART.tree` is built from both at load.
- The backdrop is fetched **separately with `crossOrigin` and a `?cors=1` query**. Without that, the browser reuses the non-CORS cached copy from the normal background load, which taints the canvas and makes `toBlob()` throw. `warmCardBackdrop()` prefetches it when the result renders so saving feels instant. Verified `toDataURL` succeeds, i.e. untainted.
- Export path: `navigator.share({files})` when available (on iOS this opens the share sheet → Photos or a story), otherwise an `<a download>` fallback. A cancelled share is treated as success so it doesn't also download.
- `SHARE_URL` is a plain constant — update it if the site ever moves to a custom domain.
- Absolute CDN URLs (not relative paths) are deliberate: `index.html` stays portable and works from any host, a preview tool, or emailed to Bam.

## Design language (locked decisions)
- Questions are **pure garden metaphor** — options must never contain real-world words that reveal which answer is "good" (validity/social-desirability decision). The real-life mapping lives ONLY in the narration before each question (e.g., "the sunlight is what wakes you each day").
- Story premise: "this garden is planted with your days; every corner reflects your life."
- Result copy (summaries, cute quotes, key messages) is Bam-approved — don't rewrite without being asked. This is why the emoji in the garden names (`🌸 Blooming Garden`) and cute quotes survived the emoji cleanup. The garden name is **also written to the Google Sheet**, so changing it mid-collection would split the data into two formats — needs a deliberate decision, not a styling pass.
- Art direction locked: naïve folk-art gouache storybook style (reference: soft blue/green/cream/terracotta, visible brushstrokes, no people, portrait 9:16). Constant prompt string exists; per-scene art is PENDING generation. Backgrounds bg0–bg6 currently all share one placeholder image intentionally.
- **No emoji in the visual layer.** Emoji render as glossy platform artwork (especially Apple's on iPad) and clash with gouache. Everything decorative is drawn instead: `KEEPER_IMG`, `treeArt()`, `flowerArt()`, `soilArt` are inline SVG via the `svgUri()` helper, the list bullets are CSS shapes, and the mute button uses `ICON_SOUND`/`ICON_MUTED` SVG. All use literal palette hex (SVG strings can't read `var()`). Emoji **do** remain in copy — the story tips (📱/🔊) and the Bam-approved garden names and quotes — deliberately, see below.
- Thai face is **Mali** (handwritten, warm), English stays **Quicksand**. Sarabun was replaced because it's Thailand's government-standard document face and made a storybook feel like a form. Mali runs ~20% wider than Sarabun; every screen was checked for overflow after the swap.
- Ambient effect is **drifting petals**, not falling leaves: `.petal` falls, nested `.sway` sways and rotates. The two axes are the point — a single `translateY` reads as dropping stones. Negative `animation-delay` spreads them on load. Respects `prefers-reduced-motion`.
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
- `index.html` is UTF-8 **without BOM**, ~6,400 Thai codepoints. The **repo** stores LF; the Windows **working copy** is CRLF because `core.autocrlf` rewrites it on checkout (harmless in the browser, but don't be surprised by the 1,296-byte difference between local and served). Windows PowerShell 5.1 reads files as ANSI by default and will mangle the Thai — always force UTF-8 (`[System.IO.File]::ReadAllBytes` + a strict `UTF8Encoding`), and keep .ps1 helper scripts pure-ASCII or PS 5.1 mis-parses them too.
- The MP3 is 3.9MB at 112kbps — already compressed; don't recompress further without asking.
- When testing the walk end-to-end, **stub `window.fetch` first** or the result screen posts junk rows to the live Google Sheet.

## Known pending threads
1. Gouache art generation (owner + Bam) → then build the background zoom/tint system. **Owner's stated plan: ALL current drawn-SVG art (keeper, 4 trees, 3 flowers, soil) gets replaced by AI-generated images**, not just the backgrounds. The drawn SVG is a stopgap that replaced emoji, not a destination.
   - Backgrounds already have a CDN loader. The scene art (`KEEPER_IMG`, `ART.tree`, `ART.flower`, `soilArt`) does **not** — those are still inline data-URIs. Generated images must go to `/assets` + CDN too, or the file re-bloats back toward the 5MB problem that was just fixed.
   - The scene art is layered (`.tree`/`.fl` sit over `.soil` and the background), so it needs **transparent** PNG/WebP. Most image generators return opaque rectangles — that will need alpha or background removal, or the trees appear as pasted boxes.
   - The 4 seasonal trees should read as *the same tree* in 4 seasons. Generate one and vary it (same seed/reference) rather than four independent prompts.
2. ~~Hosting~~ **DONE** — GitHub Pages, live at https://hlyjezz.github.io/Growth-of-garden/ (Settings → Pages → `main` / root). Every push to `main` redeploys automatically, and the same push updates the jsDelivr assets. This is the URL for iPad Safari testing. Because asset URLs are absolute, `index.html` still works from anywhere without carrying a folder alongside it.
3. An old "music not playing" report was likely device mute / preview-tool artifact; music logic is verified correct.
