# brianfountain.com

The source for [brianfountain.com](https://brianfountain.com), the personal site of Brian Fountain. On a desktop the home page lays out two decades of work as a Venn diagram of education, technology and art. Below 900px wide it becomes a timeline.

It's plain HTML, CSS and JavaScript. There's no build step or framework to install. Each page is one self-contained `index.html` with its styles and scripts inline, and the only outside request is Google Fonts.

## What's here

| Path | Contents |
|------|----------|
| `index.html` | The home page. Every role and venture comes from the `DATA` array in its `<script>` block, which drives the Venn, the timeline and the detail panel. |
| `projects/` | Shipped projects, with a technical overview of G3NPRO under `projects/g3npro/`. |
| `work/` | Longer pages on specific kinds of work: instructional design, curriculum design, technical enablement and instruction, AI product design, AI fluency education, and creative technology. `work/journey/` is a working rebuild of Journey, a learning-in-the-feed pilot. |
| `writing/` | Essays on learning design, plus a policy sketch (Universal Rice & Beans) with its own interactive cost model. |
| `assets/` | Logos, project icons, screenshots, audio and video shared across pages. |
| `docs/` | Planning notes from building the projects page. |

## Running it locally

```bash
python3 -m http.server 8099
```

Then open http://localhost:8099. Any static file server works. Check pages at a phone width as well as a desktop one, since the home page switches layouts at 900px.

## Editing

To add or change a role on the home page, edit its entry in `DATA`:

```js
{ role:"…", company:"…", year:"…",
  x:NNN, y:NNN, w:NNN,                  // position on the 1260×600 Venn stage
  domains:["Education","Technology"],   // which circles it sits in
  accent:"#RRGGBB", link:"example.com",
  filters:["…"],
  blurb:"…", tags:["…"] }
```

Card images are mapped by company in the separate `IMG` object.

A new essay goes in its own folder under `writing/` with an `index.html`, plus a card in `writing/index.html`. If the post has its own share image, give its card the `has-img` class and a thumbnail.

## Images

Images marked with an "AI" badge on the site were generated with AI. Charts in the essays are illustrative unless they cite a source.

## Rights

The writing, images and media are © Brian Fountain. Please ask before reusing them.
