# Projects Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a self-contained `/projects` page — a cinematic vertical "filmstrip" of Brian's own projects, each leading with its visual and a single primary action.

**Architecture:** A new root-level `projects.html` mirroring `index.html`'s self-contained pattern (inline data array + inline CSS + inline JS). Projects render as stacked full-width scroll-snapping "bands" that reuse the home page's palette, fonts, and accent system. `index.html` gains a cross-link to the new page.

**Tech Stack:** Plain HTML/CSS/JS (no framework, no build). Google Fonts (Fraunces, Manrope, JetBrains Mono) as on the home page. `ffmpeg` for the one video transcode. `python3 -m http.server` for local verification.

## Global Constraints

- **Self-contained:** all data/CSS/JS inline in `projects.html`. No external data fetch. Must work on `file://` and `python3 -m http.server`.
- **Palette (verbatim from `index.html`):** field `#100d0b`; stage gradient `radial-gradient(120% 80% at 50% 30%,#1a1512,#100d0b 72%)`; ink `#efe6da`; accents `#8fd0c4` (tech), `#c9a2ec` (art), `#e7a68d` (education), `#d9c48a`, `#d9a8c4`.
- **Fonts:** `Fraunces` (display), `Manrope` (body), `JetBrains Mono` (labels) — same `<link>` as `index.html`.
- **Only external dependency is Google Fonts.** All imagery/video is local under `assets/`.
- **Commits:** the user's standing rule is *don't commit until instructed*. Each task ends with a commit step, but only run it once the user gives the go-ahead at that checkpoint.
- **Verification is visual/behavioral**, not unit tests (this repo has no test harness): local server + browser, both desktop (≥900px) and mobile widths.

---

### Task 1: Transcode the G3NPRO video

**Files:**
- Create: `assets/g3npro_sample.mp4` (transcoded, web-optimized)
- Modify: `.gitignore` (exclude the 124 MB source `.mov`)
- Source (already present, not deployed): `assets/Say_What_You_See.mov`

**Interfaces:**
- Produces: `assets/g3npro_sample.mp4` — a muted, `+faststart` H.264 MP4 consumed by the G3NPRO band's `<video>` in Task 3.

- [ ] **Step 1: Transcode with ffmpeg (muted, compressed, faststart)**

```bash
ffmpeg -y -i assets/Say_What_You_See.mov \
  -an \
  -c:v libx264 -profile:v high -pix_fmt yuv420p \
  -vf "scale='min(1280,iw)':-2" \
  -crf 30 -preset slow \
  -movflags +faststart \
  assets/g3npro_sample.mp4
```
(`-an` drops audio since the band autoplays muted+looped.)

- [ ] **Step 2: Verify output is small and valid**

Run: `du -h assets/g3npro_sample.mp4 && ffprobe -v error -show_entries stream=codec_name,width,height,duration -of default=noprint_wrappers=1 assets/g3npro_sample.mp4`
Expected: size in the low single-digit MB (target < 8 MB); `codec_name=h264`, width ≤ 1280, duration ≈ 76s. If still large, re-run with `-crf 32` or add `-r 24`.

- [ ] **Step 3: Keep the source out of git**

Add to `.gitignore`:
```
assets/Say_What_You_See.mov
```

- [ ] **Step 4: Commit** (on user go-ahead)

```bash
git add assets/g3npro_sample.mp4 .gitignore
git commit -m "chore: add web-optimized g3npro sample video"
```

---

### Task 2: Gather imagery + finalize archive/new-project content

**Files:**
- Create (as needed): `assets/shot_bender.jpeg`, `assets/shot_chapters.jpeg`, and `assets/shot_<name>.jpeg` for any archive site still live.
- Create (working note, not committed): `docs/superpowers/plans/projects-content.md` — captured blurbs + live/dead status feeding Task 3.

**Interfaces:**
- Produces: (a) screenshot files in `assets/` for live sites; (b) a content note listing, per project: `live?`, `category`, `accent`, one-line `blurb`, and `media` decision (image vs placeholder). Task 3's `PROJECTS` array consumes this verbatim.

- [ ] **Step 1: Load the browser tools**

Use ToolSearch: `select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__read_page`

- [ ] **Step 2: Capture the two unknown new sites**

Navigate a new tab to `https://bender.brianfountain.com` and `https://chapters.brianfountain.com`. For each: screenshot the hero/landing view, and read enough of the page to write a one-line blurb + pick a category and accent. Save screenshots as `assets/shot_bender.jpeg` / `assets/shot_chapters.jpeg` (downscale to ~1280px wide, quality comparable to existing `shot_*.jpeg` ~70–140 KB).

- [ ] **Step 3: Check which archive sites are still live**

Visit each: `daystolive.net`, `couchcachet.com`, `uchoos.com`, `zown` (search if domain unknown), `spoton` (StartupBus trivia — likely dead), `futuremate.us`, `armybakesale.com`. For any that load a real page, screenshot → `assets/shot_<name>.jpeg`. For dead ones, mark `media:{type:"placeholder"}`.

- [ ] **Step 4: Write the content note**

Record per-project: name, year, category, accent, blurb (use the user-provided copy for archive projects; best-effort for Bender/Chapters), tags, media decision, action (href or archived), filters. This is the source Task 3 transcribes into `PROJECTS`.

- [ ] **Step 5: Commit** (on user go-ahead)

```bash
git add assets/shot_*.jpeg
git commit -m "assets: add project screenshots for projects page"
```

---

### Task 3: Build `projects.html` — scaffold, data, and band rendering

**Files:**
- Create: `projects.html`

**Interfaces:**
- Consumes: `assets/g3npro_sample.mp4` (Task 1); screenshots + content note (Task 2).
- Produces: a working page that renders every project as a scroll-snapping band (image/placeholder), styled with the shared palette. Task 4 adds video autoplay; Task 5 adds the top bar + filters. Exposes JS globals: `PROJECTS` (array), `renderBands()`, container `#film`.

- [ ] **Step 1: Create the document skeleton**

Head mirrors `index.html` (charset, viewport, title `Projects — Brian Fountain`, canonical `https://brianfountain.com/projects`, favicon, OG/Twitter tags, the same Google Fonts `<link>`). Body:

```html
<body>
  <header class="pbar" id="pbar"><!-- Task 5 --></header>
  <main class="film" id="film"><!-- bands injected here --></main>
  <script>/* PROJECTS + render below */</script>
</body>
```

- [ ] **Step 2: Add the base + band CSS**

Inline `<style>` reusing the global tokens:

```css
*{margin:0;padding:0;box-sizing:border-box}
html{scroll-behavior:smooth}
body{background:#100d0b;color:#efe6da;font-family:'Manrope',sans-serif;-webkit-font-smoothing:antialiased}
a{color:inherit}
.film{scroll-snap-type:y proximity}
.band{position:relative;min-height:78vh;max-height:720px;display:flex;align-items:flex-end;
  overflow:hidden;scroll-snap-align:start;border-bottom:1px solid rgba(239,230,218,.08)}
.band-media{position:absolute;inset:0;background:#17130f}
.band-media img,.band-media video{width:100%;height:100%;object-fit:cover;display:block}
.band-scrim{position:absolute;inset:0;
  background:linear-gradient(90deg,rgba(16,13,11,.94),rgba(16,13,11,.55) 46%,rgba(16,13,11,.12) 100%),
             linear-gradient(0deg,rgba(16,13,11,.92),transparent 58%)}
.band-content{position:relative;z-index:2;padding:clamp(24px,5vw,64px);max-width:640px;width:100%}
.band-index{font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.18em;color:rgba(239,230,218,.5)}
.band-name{font-family:'Fraunces',serif;font-optical-sizing:auto;font-size:clamp(34px,6vw,70px);line-height:1.02;margin-top:8px}
.band-meta{font-family:'JetBrains Mono',monospace;font-size:12px;letter-spacing:.06em;margin-top:10px}
.band-blurb{font-size:clamp(14px,1.4vw,16px);line-height:1.55;color:rgba(239,230,218,.82);margin-top:12px;max-width:560px}
.band-tags{display:flex;flex-wrap:wrap;gap:7px;margin-top:16px}
.band-tags span{padding:4px 11px;border-radius:100px;border:1px solid rgba(239,230,218,.2);
  font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.03em;color:rgba(239,230,218,.72)}
.band-cta{display:inline-flex;align-items:center;gap:8px;margin-top:20px;padding:11px 18px;border-radius:100px;
  border:none;font-family:'JetBrains Mono',monospace;font-size:11px;letter-spacing:.06em;font-weight:700;
  color:#12110f;cursor:pointer;text-decoration:none;transition:.2s}
.band-cta:hover{filter:brightness(1.08)}
.band-archived{display:inline-block;margin-top:20px;padding:8px 14px;border-radius:100px;
  border:1px dashed rgba(239,230,218,.25);font-family:'JetBrains Mono',monospace;font-size:10px;
  letter-spacing:.1em;color:rgba(239,230,218,.5)}
.ph{position:absolute;inset:0;display:flex;align-items:center;justify-content:center}
.ph-mono{font-family:'Fraunces',serif;font-size:clamp(60px,14vw,180px);opacity:.22;letter-spacing:.02em}
@media (max-width:640px){ .band{min-height:70vh} }
```
(CTA background + placeholder gradient are set per-band from `accent` in JS.)

- [ ] **Step 3: Add the `PROJECTS` data array**

Transcribe the Task 2 content note. Recent first, then archive. Example entries (fill all 13):

```js
var PROJECTS = [
  { name:"G3NPRO", year:"2025", cat:"AI · Animation", accent:"#8fd0c4",
    blurb:"A platform giving enterprise animation studios generative-AI tools that accelerate production and unlock new creative possibilities.",
    tags:["Generative AI","Animation","Product"],
    media:{type:"video", src:"assets/g3npro_sample.mp4", poster:"assets/shot_g3npro.jpeg"},
    action:{label:"Visit g3npro.com", href:"https://g3npro.com"},
    filters:["Recent","AI","Founder"] },
  { name:"GenBuzz", year:"2025", cat:"AI · News", accent:"#8fd0c4",
    blurb:"An agentically-driven news aggregator for the fast-moving world of AI filmmaking — surfacing what matters without the noise.",
    tags:["Agentic","News","AI film"],
    media:{type:"image", src:"assets/shot_genbuzz.jpeg"},
    action:{label:"Open genbuzz.news", href:"https://genbuzz.news"},
    filters:["Recent","AI","Founder","Story"] },
  // Bender, 1K Notes, Chapters, Vibeside, then archive: DaysToLive, CouchCachet,
  // uChoos, zown, SpotOn Trivia, FutureMate, ArmyBakeSale — from the Task 2 note.
];
```

- [ ] **Step 4: Write `renderBands()` (image + placeholder + CTA/archived)**

```js
function el(t,c,x){var e=document.createElement(t);if(c)e.className=c;if(x!=null)e.textContent=x;return e;}
function renderBands(){
  var film=document.getElementById("film"); film.innerHTML="";
  var total=String(PROJECTS.length).padStart(2,"0");
  PROJECTS.forEach(function(p,i){
    var band=el("div","band"); band.dataset.filters=(p.filters||[]).join(",");
    var media=el("div","band-media");
    if(p.media.type==="image"){ var img=el("img"); img.src=p.media.src; img.alt=p.name; img.loading="lazy"; media.appendChild(img); }
    else if(p.media.type==="placeholder"){ media.style.background="radial-gradient(120% 120% at 30% 20%,"+p.accent+"33,#100d0b 70%)";
      var ph=el("div","ph"); var mono=el("div","ph-mono",p.name); mono.style.color=p.accent; ph.appendChild(mono); media.appendChild(ph); }
    // video handled in Task 4
    band.appendChild(media);
    band.appendChild(el("div","band-scrim"));
    var c=el("div","band-content");
    c.appendChild(el("div","band-index",String(i+1).padStart(2,"0")+" / "+total));
    c.appendChild(el("div","band-name",p.name));
    var meta=el("div","band-meta",p.year+"  ·  "+p.cat); meta.style.color=p.accent; c.appendChild(meta);
    c.appendChild(el("div","band-blurb",p.blurb));
    var tg=el("div","band-tags"); (p.tags||[]).forEach(function(t){ tg.appendChild(el("span",null,t)); }); c.appendChild(tg);
    if(p.action){ var a=el("a","band-cta",p.action.label); a.href=p.action.href; a.target="_blank"; a.rel="noopener";
      a.style.background=p.accent; c.appendChild(a); }
    else { c.appendChild(el("div","band-archived","ARCHIVED")); }
    band.appendChild(c);
    film.appendChild(band);
  });
}
renderBands();
```

- [ ] **Step 5: Verify in the browser**

Run: `python3 -m http.server 8099` then open `http://localhost:8099/projects.html`.
Expected: all bands render top-to-bottom, screenshots fill each band with readable overlaid text, placeholders show accent monograms, CTAs open the right sites, scroll snaps between bands. Check desktop and a narrow (~390px) width.

- [ ] **Step 6: Commit** (on user go-ahead)

```bash
git add projects.html
git commit -m "feat: add projects filmstrip page (bands, data, rendering)"
```

---

### Task 4: Video band — autoplay-in-view with reduced-motion fallback

**Files:**
- Modify: `projects.html` (extend `renderBands()` + add an IntersectionObserver)

**Interfaces:**
- Consumes: `renderBands()` and `PROJECTS` (Task 3).
- Produces: G3NPRO's `media.type==="video"` renders a muted looped inline `<video>` that plays only while ≥50% visible; pauses otherwise; falls back to poster on `prefers-reduced-motion`.

- [ ] **Step 1: Render the `<video>` element in the media branch**

In `renderBands()`, add alongside the image/placeholder branches:
```js
else if(p.media.type==="video"){
  var v=el("video"); v.src=p.media.src; v.poster=p.media.poster||"";
  v.muted=true; v.loop=true; v.playsInline=true; v.setAttribute("playsinline","");
  v.preload="metadata"; v.dataset.autoplay="1"; media.appendChild(v);
}
```

- [ ] **Step 2: Add the play/pause observer after `renderBands()`**

```js
(function(){
  var reduce=window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  var vids=document.querySelectorAll('video[data-autoplay]');
  if(reduce || !("IntersectionObserver" in window)) return; // poster stays, no autoplay
  var io=new IntersectionObserver(function(entries){
    entries.forEach(function(e){
      var v=e.target;
      if(e.isIntersecting && e.intersectionRatio>=0.5){ v.play().catch(function(){}); }
      else { v.pause(); }
    });
  },{threshold:[0,0.5,1]});
  vids.forEach(function(v){ io.observe(v); });
})();
```

- [ ] **Step 3: Verify**

Reload `http://localhost:8099/projects.html`. Expected: the G3NPRO band shows the poster, then plays (silently, looping) when scrolled into view and pauses when scrolled away. Toggle OS "Reduce Motion" → only the poster shows. Confirm on a mobile width the video still fills the band.

- [ ] **Step 4: Commit** (on user go-ahead)

```bash
git add projects.html
git commit -m "feat: autoplay g3npro video band when in view (reduced-motion safe)"
```

---

### Task 5: Sticky top bar — back-link + category filters

**Files:**
- Modify: `projects.html` (`.pbar` markup, CSS, filter JS)

**Interfaces:**
- Consumes: `#film` bands with `data-filters` (Task 3).
- Produces: a sticky bar with a `↩ Brian Fountain` link to `/` and chips `All · Recent · AI · Games · Story · Founder` that show/hide bands by `data-filters`.

- [ ] **Step 1: Add the bar CSS**

```css
.pbar{position:sticky;top:0;z-index:10;display:flex;align-items:center;justify-content:space-between;
  gap:14px;padding:12px clamp(16px,4vw,40px);background:rgba(14,11,9,.82);
  backdrop-filter:blur(9px);-webkit-backdrop-filter:blur(9px);border-bottom:1px solid rgba(239,230,218,.1)}
.pback{display:inline-flex;align-items:center;gap:8px;font-family:'Fraunces',serif;font-size:18px;text-decoration:none}
.pback span{font-family:'JetBrains Mono',monospace;font-size:12px;opacity:.7}
.pfilters{display:flex;flex-wrap:wrap;gap:7px}
.pchip{padding:5px 12px;border-radius:100px;border:1px solid rgba(239,230,218,.2);background:rgba(239,230,218,.04);
  color:rgba(239,230,218,.75);font-family:'JetBrains Mono',monospace;font-size:10px;letter-spacing:.04em;cursor:pointer;transition:.2s}
.pchip.on{border-color:transparent;background:#efe6da;color:#12110f;font-weight:700}
@media (max-width:640px){ .pfilters{overflow-x:auto;flex-wrap:nowrap;-webkit-overflow-scrolling:touch} .pchip{flex:none} }
```

- [ ] **Step 2: Fill the `.pbar` markup + wire filters in JS**

```html
<header class="pbar" id="pbar">
  <a class="pback" href="/">↩ <span>Brian Fountain</span></a>
  <nav class="pfilters" id="pfilters"></nav>
</header>
```
```js
var FILTERS=["All","Recent","AI","Games","Story","Founder"];
var activeFilter="All";
(function buildFilters(){
  var box=document.getElementById("pfilters");
  FILTERS.forEach(function(f){
    var c=el("button","pchip"+(f==="All"?" on":""),f); c.dataset.f=f;
    c.onclick=function(){ activeFilter=f; applyFilter(); };
    box.appendChild(c);
  });
})();
function applyFilter(){
  document.querySelectorAll("#pfilters .pchip").forEach(function(c){ c.classList.toggle("on",c.dataset.f===activeFilter); });
  document.querySelectorAll("#film .band").forEach(function(b){
    var show=activeFilter==="All" || (b.dataset.filters||"").split(",").indexOf(activeFilter)!==-1;
    b.style.display=show?"":"none";
  });
}
```
(For local `file://` testing the back-link `/` won't resolve; that's fine — it resolves once served at the site root. Optionally use `index.html` during local checks.)

- [ ] **Step 3: Verify**

Reload. Expected: sticky bar stays pinned while scrolling; clicking a chip shows only matching bands and re-highlights; `All` restores everything; on narrow width the chips scroll horizontally. Back-link points home.

- [ ] **Step 4: Commit** (on user go-ahead)

```bash
git add projects.html
git commit -m "feat: sticky bar with back-link and category filters"
```

---

### Task 6: Cross-link from `index.html`

**Files:**
- Modify: `index.html` (desktop `.nav-row` cluster ~line 189–196; mobile `.m-top` ~line 261–267)

**Interfaces:**
- Consumes: the deployed `/projects` route.
- Produces: a `Projects ↗` link on the home page in both desktop and mobile headers, styled with the existing `.btn-ghost` treatment.

- [ ] **Step 1: Add the desktop link**

In `index.html`, inside the desktop control cluster (the `<div style="display:flex;gap:10px;align-items:center">` next to the seg toggle), add before the Play button:
```html
<a href="/projects" class="btn-ghost" style="text-decoration:none"><span style="font-size:12px">◲</span><span>PROJECTS</span></a>
```

- [ ] **Step 2: Add the mobile link**

In the `.m-top` header, add a compact link next to `.m-seg`:
```html
<a href="/projects" class="m-chip" style="text-decoration:none;flex:none">PROJECTS ↗</a>
```
(Or place it in `.m-top` styled like `.btn-ghost`; keep it visually consistent with the mobile header.)

- [ ] **Step 3: Verify**

Reload `http://localhost:8099/index.html` at desktop and mobile widths. Expected: a Projects link appears in both headers and (once served) navigates to `/projects`. Confirm it doesn't disturb the existing nav layout.

- [ ] **Step 4: Commit** (on user go-ahead)

```bash
git add index.html
git commit -m "feat: link home page to projects filmstrip"
```

---

### Task 7: Final verification pass

**Files:** none (verification only)

- [ ] **Step 1: Full desktop pass**

Serve with `python3 -m http.server 8099`, open `/projects.html` at ≥900px. Walk every band: image quality, text contrast, video autoplay/pause, placeholders, each CTA opens the correct site in a new tab, scroll-snap feels right, filters work, back-link + `index.html` cross-link work.

- [ ] **Step 2: Mobile pass**

Resize to ~390px (or device emulation). Confirm bands reflow, content stays legible, filter chips scroll, video fills its band, no horizontal overflow.

- [ ] **Step 3: Asset + link sanity check**

Run: `grep -oE "assets/[A-Za-z0-9_./-]+\.(jpeg|jpg|png|svg|mp4)" projects.html | sort -u | while read f; do [ -f "$f" ] && echo "OK $f" || echo "MISSING $f"; done`
Expected: every referenced asset prints `OK`.

- [ ] **Step 4: Confirm the source .mov is not tracked**

Run: `git status --porcelain assets/Say_What_You_See.mov`
Expected: no output (ignored/untracked).

---

## Self-Review

**Spec coverage:** File/routing → Task 3 + Caddy note (hosting out-of-band). Cross-linking → Task 6. Data model → Task 3 Step 3. Band anatomy → Task 3 Steps 2/4. Video handling → Task 1 + Task 4. Filters/top bar → Task 5. Content list → Task 2 + Task 3 Step 3. Assets (capture + placeholders + video) → Tasks 1–2. Responsive/testing → Tasks 3,5,7. All spec sections covered.

**Placeholder scan:** No "TBD/TODO" left as work-avoidance. The only deferred content (Bender/Chapters blurbs, which archive sites are live) is a defined **output of Task 2** consumed by Task 3 — an explicit interface, not a lazy gap.

**Type/name consistency:** `PROJECTS`, `renderBands()`, `el()`, `#film`, `.band`, `data-filters`, `applyFilter()`, `activeFilter`, `FILTERS` are used consistently across Tasks 3–5. Media types (`image`/`video`/`placeholder`) and `action`/`filters` field names match the spec and each other.
