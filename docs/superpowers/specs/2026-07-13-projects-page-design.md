# Projects Page — Design Spec

**Date:** 2026-07-13
**Feature:** A new `/projects` page for brianfountain.com — a visual, cinematic
"filmstrip" of the projects Brian has *created* (as distinct from the work-history
Venn/timeline on the home page). Optimized for quick access to each project or its output.

## Goals

- A dedicated, **more visual** page focused only on Brian's own projects/ventures.
- **Quick access to the project or its output** — every project leads with its visual
  and a single primary action (visit the live site, watch the video, etc.).
- Reads as the same site: reuse the home page's palette, fonts, and accent system.
- Fully **self-contained** (no build step, no external data fetch) — consistent with
  the existing single-file `index.html`.

## Non-goals

- No work-history / employment content (that stays on the home page).
- No CMS, no backend, no separate data-fetch. Data is inline.
- No unrelated refactor of `index.html` beyond adding the cross-link.

## Architecture

### File & routing
- New **`projects.html`** at repo root, fully self-contained: inline data array +
  inline CSS + inline JS, mirroring `index.html`'s structure.
- Served at **`/projects`** via a Caddy `try_files` rule mapping `/projects` →
  `projects.html`. (Hosting is out-of-band/private; this spec only notes the mapping.
  The committed change is the `projects.html` file + the home-page cross-link.)
- Works locally on `file://` and `python3 -m http.server 8099`.

### Cross-linking
- Add a `Projects ↗` link to `index.html`:
  - Desktop: in the `.nav-row` control cluster (near the Venn/Timeline segment).
  - Mobile: in the `.m-top` header.
- `projects.html` has a `↩ Brian Fountain` back-link to `/` in its sticky top bar.
- Shared header styling (same fonts, ghost-button treatment).

## Data model

A single inline `PROJECTS` array is the source of truth. Each entry drives one band:

```js
{ name:"G3NPRO", year:"2025", cat:"AI · Animation", accent:"#8fd0c4",
  blurb:"A platform giving enterprise animation studios generative-AI tools that
         accelerate production and unlock new creative possibilities.",
  tags:["Generative AI","Animation","Product"],
  media:{ type:"video", src:"assets/g3npro_sample.mp4", poster:"assets/shot_g3npro.jpeg" },
  action:{ label:"Visit g3npro.com", href:"https://g3npro.com" },
  filters:["Recent","AI","Founder"] }
```

Fields:
- `name`, `year`, `cat` (short category string, e.g. `"AI · Animation"`), `accent` (hex).
- `blurb` — one to two sentences.
- `tags` — short chips.
- `media` — one of:
  - `{type:"image", src, fit}` — screenshot cover (fit defaults to `cover`).
  - `{type:"video", src, poster}` — native `<video>`, muted + looped + `playsinline`,
    autoplays when scrolled into view (IntersectionObserver), pauses when out.
  - `{type:"placeholder", mono}` — accent gradient + large monogram/title for defunct
    sites with no live capture.
- `action` — primary CTA `{label, href}`; omit for archived projects (renders a muted
  `Archived` chip instead of a button).
- `filters` — which category chips select it (subset of the filter vocabulary).

## Layout & interaction

### The band
Each project renders as a **full-width vertical band**, stacked in a scroll-snapping
column (CSS scroll-snap for a cinematic filmstrip feel):
- Band height ≈ `78vh`, capped ≈ `720px`; min height ensures readability on short
  viewports.
- Media fills the band; a **left-to-dark gradient scrim** guarantees text contrast.
- **Content column, overlaid bottom-left:** large index `NN / TT`, `name` in Fraunces,
  `year · cat` in JetBrains Mono (accent-colored), `blurb`, tag chips, accent CTA button.
- Palette, fonts, and accent colors are taken directly from `index.html`
  (`#100d0b` field, `#efe6da` ink; domain accents `#8fd0c4` tech, `#c9a2ec` art,
  `#e7a68d` education, plus per-project accents).

### Video handling (G3NPRO)
- Source asset provided by user: `assets/Say_What_You_See.mov` (124 MB, 1280×720, ~76s).
- **Transcode** to web-optimized `assets/g3npro_sample.mp4` with ffmpeg:
  H.264, `-movflags +faststart`, **muted (drop audio)** since it autoplays looped,
  compressed to a few MB (CRF ~28–30, cap width 1280). Original `.mov` is not deployed
  (kept out via `.gitignore` or simply not referenced/committed).
- Band uses `<video muted loop playsinline poster="assets/shot_g3npro.jpeg">`; JS
  IntersectionObserver plays/pauses based on visibility. Respects
  `prefers-reduced-motion` (no autoplay → poster only, click opens g3npro.com).

### Filters & top bar
- Slim **sticky top bar**: `↩ Brian Fountain` back-link on the left; category filter
  chips on the right (`Recent · AI · Games · Story · Founder`).
- Selecting a chip filters which bands are shown (toggle off to clear), mirroring the
  home page's filter behavior. "All" is the default (no chip active = show everything).

### Responsive
- Same single-file responsive approach as `index.html`. Bands reflow for mobile:
  content column stays bottom-left, font sizes step down, sticky bar compacts.
- Test both desktop (≥900px) and mobile widths.

## Content — projects in order (recent → archive)

**Recent (2025):**
1. **G3NPRO** — g3npro.com — AI · Animation — *video* (`g3npro_sample.mp4`).
2. **GenBuzz** — genbuzz.news — AI · News — screenshot (`shot_genbuzz.jpeg`).
3. **Bender** — bender.brianfountain.com — *TBD, capture + best-effort blurb*.
4. **1K Notes** — 1knotes.com — Music · AI — screenshot (`shot_1knotes.jpeg`).
5. **Chapters** — chapters.brianfountain.com — *TBD, capture + best-effort blurb*.
6. **Vibeside** — vibesi.de — Code · AI — screenshot (`shot_vibeside.jpeg`).

**Archive:**
7. **DaysToLive** — DaysToLive.net (2015) — SMS mortality reminders. Product design,
   information architecture.
8. **CouchCachet** — (2013) — social-satire app built in 36h; won the Mashery Global
   prize; 15M+ media reach (NYT, Buzzfeed, Mashable, ABCnews).
9. **uChoos** — (2011) — interactive-fiction / choose-your-own-adventure platform for
   feature phones.
10. **zown** — (2011) — outdoor team strategy game (capture-the-flag on steroids);
    official selection, 2012 Come Out and Play & City of Play festivals.
11. **SpotOn Trivia** — (2011) — conceived, built and launched over the 48h StartupBus
    ride NYC → Austin.
12. **FutureMate** — (2011) — post-apocalyptic online-dating transmedia story; won the
    first StoryCode × Lincoln Film Center story hackathon (36h, team of four); press in
    Forbes, Washington Post, PBS.
13. **ArmyBakeSale** — ArmyBakeSale.com (2003) — comedic site imagining a world where
    schools are funded and the Army holds a bake sale to buy a bomber.

## Assets

- **Auto-capture** (browser screenshots) for live sites lacking a `shot_*`: Bender,
  Chapters, and any archive sites still up (e.g. DaysToLive). Reuse existing
  `shot_genbuzz.jpeg`, `shot_1knotes.jpeg`, `shot_vibeside.jpeg`, `shot_g3npro.jpeg`.
- **Styled placeholders** for defunct sites (accent gradient + monogram): ArmyBakeSale
  and any archive project whose site is down.
- **Video:** transcode `Say_What_You_See.mov` → `g3npro_sample.mp4` (see above).

## Open items / risks

- **Bender & Chapters** content is unknown; blurbs/categories are best-effort from the
  live captures and flagged for the user to refine.
- Which archive sites are still live is determined during implementation (browser check);
  live → screenshot, dead → placeholder.
- Large video: must confirm the transcoded mp4 is small enough (target a few MB) and
  plays inline on iOS Safari (`playsinline`, muted).

## Testing / verification

- `python3 -m http.server 8099`, open `http://localhost:8099/projects.html`.
- Verify desktop (≥900px) and mobile widths.
- Confirm: filmstrip scroll-snap, video autoplay-in-view + pause-out + reduced-motion
  fallback, filter chips, every CTA opens the right target, cross-links both ways.
