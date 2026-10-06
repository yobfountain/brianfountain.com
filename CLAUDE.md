# brianfountain.com

Brian Fountain's personal site — a single self-contained `index.html` presenting his
career as an interactive **Venn diagram** (Education / Technology / Art) on desktop that
collapses to a **chronological timeline** on mobile.

## Repository layout

| Path | What it is |
|------|-----------|
| `index.html` | The entire site — HTML + CSS + JS inline. The `DATA` array (in the `<script>`) is the single source of truth for every role/venture. |
| `brian_fountain.jpeg` | Avatar shown at the center of the Venn ("BRIAN") and used as the OG/Twitter share image. |
| `assets/` | Card imagery + `favicon.svg` (BF monogram), all local (self-contained). `logo_*` = company logos, `icon_*` = project favicons used as chips, `shot_*` = live-site screenshots used in the big preview. |
| `work/journey/` | Supplemental page recreating the Journey pilot at TikTok: an interactive phone demo (lesson, knowledge check, stitch prompt, sticker board) beside prose. Self-contained; its still lives in `work/journey/assets/`. |
| `work/instructional-design/` | Portfolio page collecting the roles that involved educational content (TikTok, Thinkful, Google, NYCDA, General Assembly), the facilitation work, and the Journey prototype. Self-contained; reuses `assets/` logos and the Journey still. |
| `work/ai-product-design/` | Consolidated portfolio of the products built on generative models (G3NPRO, GenBuzz, Vibeside, 1K Notes, Chapters). Each opens with a three-cell strip stating what the model does, what the person does, and what the interface owes them. Self-contained; reuses `assets/` screenshots and the G3NPRO demo video. |
| `work/creative-technologist/` | Portfolio of the art-plus-code work: the browser games, the Corn for Cars data story, the generative tools for makers, JC2K and the Outside Lands song, and a 2003–2015 archive of hackathon and transmedia projects. Self-contained; reuses `assets/` screenshots, audio and video. |

Fonts (Fraunces, Manrope, JetBrains Mono) load from Google Fonts — the only external
dependency. Everything else is bundled.

## Editing the site

All content lives in the `DATA` array near the top of the `<script>` block in `index.html`.
Each entry drives a Venn node, a timeline row, and the detail panel:

```js
{ role:"…", company:"…", year:"…",
  x:NNN, y:NNN, w:NNN,            // Venn position (center-based) within the 1260×600 stage; w = chip width override
  domains:["Education","Technology"],  // which circles it sits in
  accent:"#RRGGBB", link:"example.com", // link makes the preview clickable
  filters:["…"],                 // which filter chips select it
  blurb:"…", tags:["…"] }
```

Card imagery is mapped separately in the `IMG` object (keyed by `company`):
`ar:"square"` logos render in a right-aligned square frame, `ar:"wide"` screenshots in a
16:9 frame — the preview slot is fixed-width so the detail text never shifts. Projects also
get a `chip:` (square favicon) used for the small Venn/timeline thumbnails while the big
preview shows the full `src:` screenshot.

### Verifying changes locally

```bash
python3 -m http.server 8099    # then open http://localhost:8099/index.html
```

The Venn is desktop-only (`min-width:900px`); below that it forces the mobile timeline.
Test both widths.

## Deploy & hosting

The site is static (updating = copying files). Deployment, hosting, and DNS are handled
out-of-band and documented **privately** — they are intentionally kept out of this public
repo.

## Conventions

- Don't offer to commit changes until asked.
- In Ruby/Rails work: enums as `enum :status, [:draft, :for_approval, :published, :hidden]`;
  don't embed ERB in HTML attributes — use the `<%= tag.div %>` form.
