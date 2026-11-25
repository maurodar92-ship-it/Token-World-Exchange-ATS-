# TWE & NFT Foundry Website: Complete Design and Code

Creator field: Mauro Lomibao Dar III

This document describes how every page of the site is designed, how the pages are generated, and then lists the complete source code. The site is a static, multi-page website. It is not a single-page app, and it has no server or database. A Node.js script (`build.mjs`) writes plain HTML, CSS, SVG and JSON files. Any static host can serve them.

---

## 1. Overview

| Item | Value |
|---|---|
| Purpose | Documentation and a generated catalogue for Token World Exchange ATS (TWE) and the Standard Super NFT Foundry |
| Type | Static site, generated at build time |
| Full-size output | 6,850 pages: 3,612 NFT editions, 3,000 token concepts, 238 hub, gallery, trait and documentation pages |
| Languages | Node.js (ES modules) to generate; HTML5, CSS3 and about 20 lines of vanilla JavaScript (search) in the output |
| Dependencies | None at runtime or build time (Node 22+ only) |
| Engine | The real `engine/foundry.js` from the Standard Super NFT Foundry (copied to `vendor/`) |
| Deployed project | Manus managed project "TWE and NFT Foundry" (`twenft`) |

Scale is controlled by environment variables:

| Variable | Default | Meaning |
|---|---|---|
| `COSMIC_EDITIONS` | 3000 | Editions in the Infinite Horizon Cosmic Nexus collection |
| `MATRIX_EDITIONS` | 600 | Editions in Matrix Heroes (trait space allows about 900 at most) |
| `TITAN_EDITIONS` | 12 | Editions in Mythic Titans (trait space is exactly 16) |
| `TOKENS` | 3000 | Token concept pages |
| `SITE_URL` | empty | If set, writes an absolute `sitemap.xml` and `robots.txt` |
| `OUT_DIR` | `./dist` | Output folder |

Note on the live version: while diagnosing a publishing problem, the committed `dist/` was rebuilt at a smaller size (300 / 100 / 300). Running the build with the defaults above restores the full 6,850 pages.

---

## 2. Design system

### 2.1 Design direction

The site reuses the TWE "Signal Terminal" look: a dark, technical interface meant for dense data. The documentation therefore feels like part of the product. Nothing is decorative for its own sake; every colour carries a meaning.

### 2.2 Colour palette

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#07131f` | Page canvas (deep navy) |
| `--panel` | `#0d1f30` | Cards, header, footer, tables |
| `--panel2` | `#10283d` | Inline code, bars, hover surfaces |
| `--line` | `#1d3a52` | 1px separators and borders |
| `--text` | `#e6f1f8` | Body text |
| `--mute` | `#8fa9bc` | Secondary text, breadcrumbs, lead paragraphs |
| `--cy` | `#20d9ff` | Primary accent: links, stats, focus, active page |
| `--gn` | `#36d7a1` | Positive accent: token tickers, banner text |
| `--co` | `#ff7184` | Negative accent (reserved) |
| `--gold` | `#e8c26a` | "Legendary" tier and gold art accents |
| Banner | `#0b2a22` on `#9be8cc` | Always-visible sandbox disclaimer |
| Notice | `#2a1a22` on `#ffc2cd` | "Concept only" warnings |

Rarity tier colours: Mythic `#ff7be5`, Legendary `#e8c26a`, Epic `#b58cff`, Rare `#20d9ff`, Common `#8fa9bc`.

### 2.3 Typography

| Role | Font | Notes |
|---|---|---|
| Headings and body | Space Grotesk, then system-ui, Segoe UI, Roboto | 16px base, line-height 1.65 |
| Numbers, tickers, IDs, code | IBM Plex Mono, then ui-monospace, Menlo, Consolas | Keeps digits aligned in tables |
| H1 | `clamp(1.7rem, 4vw, 2.5rem)`, line-height 1.15 | Fluid size |
| H2 | 1.3rem with a bottom hairline | Section divider |
| Lead paragraph | 1.12rem, `--mute`, max width 760px | Page summary under the H1 |

The fonts are named in the CSS stack but not downloaded, so pages stay fast and work offline. If the fonts are installed they are used; otherwise the system fallbacks apply.

### 2.4 Layout

* Content column: `max-width: 1100px`, centred, 16px side padding, mobile-first.
* Sticky top header (panel colour, 1px bottom border) with the logo, a wordmark "TWE & NFT Foundry" and a wrapping text navigation (Home, TWE, Foundry, Collections, Tokens, Search, FAQ).
* A one-line sandbox banner sits above the header on every page.
* Footer: four link columns (TWE, Foundry, Browse, About) in an auto-fit grid, plus a fine-print disclaimer line.
* Home hero: one column on phones; at 760px and wider it becomes two columns (1.3fr text, 1fr logo).
* Edition page: image first on phones; two equal columns at 800px and wider.

### 2.5 Components

| Component | CSS class | Behaviour |
|---|---|---|
| Breadcrumbs | `.crumbs` | Small muted trail ending in the current page |
| Stat tile | `.stat` | Large mono number in cyan, small caption |
| Card | `.card` | Panel, rounded 12px, border turns cyan on hover |
| Thumbnail grid | `.grid`, `.thumb` | `repeat(auto-fill, minmax(140px, 1fr))`, square lazy-loaded SVG, caption with tier badge |
| Token card | `.tokgrid`, `.tok` | Name in bold, mono ticker in green, category beneath |
| Rarity badge | `.badge .t-<Tier>` | Pill with tier colour |
| Pager | `.pager` | Prev / numbers / Next with ellipsis; current page filled cyan |
| Table | `table` | Hairline row separators, muted header |
| Rarity bar | `.bar` | 8px bar showing trait share |
| Notice | `.notice` | Coral-tinted box for "concept only" warnings |
| Directory | `.dir` | CSS multi-column list for A to Z directories |
| Search box | `input.s` | Large input; results rendered live |

### 2.6 Responsive behaviour

All sizes use `auto-fill`/`auto-fit` grids, `clamp()` and `max-width`, so there are no device-specific breakpoints except the two (760px hero, 800px edition). The header navigation wraps. Thumbnails use `loading="lazy"`.

### 2.7 Accessibility

* Single H1 per page, then ordered H2s.
* Breadcrumb `<nav aria-label="Breadcrumb">` and pager `aria-label`.
* Thumbnails carry descriptive `alt` text; decorative gallery thumbnails use empty alt.
* Contrast: body text `#e6f1f8` on `#07131f` is well above WCAG AA; muted text `#8fa9bc` on the same background is about 7:1.
* Plain HTML links, no scripting required for navigation.

---

## 3. Page types and URL structure

All links are written as site-root paths in the source (for example `/foundry/index.html`) and are converted to relative links at write time (`../` repeated by depth). This lets the site work from any sub-path or even from a local folder. Every link points at an explicit `.html` file, never a bare folder.

| Page type | URL pattern | Count (full size) | Contents |
|---|---|---|---|
| Home | `/index.html` | 1 | Hero with logo, four stat tiles, four cards, 24 sample editions |
| TWE docs | `/twe/*.html` | 12 | Overview, sandbox boundary, look, paper trading, token concepts, NFT planner, architecture workspace, tech stack, repository map, run locally, AI Studio import, production readiness |
| Foundry docs | `/foundry/*.html` | 13 | Overview, how it works, determinism, rarity, exclusions, SVG, metadata, API, HashLips, scaling, compliance gate, testing, collections hub |
| About and legal | `/about.html`, `/faq.html`, `/glossary.html`, `/roadmap.html`, `/legal/disclaimer.html` | 5 | Static text |
| Collection overview | `/foundry/<collection>/index.html` | 3 | Stats, layer table, exclusion rules, 12 samples |
| Rarity table | `/foundry/<collection>/rarity.html` | 3 | Every trait with count, percent and bar |
| Gallery | `/foundry/<collection>/page-<n>.html` | 61 | 60 thumbnails per page |
| Trait | `/foundry/traits/<collection>/<layer>--<option>.html` | 85 | Share of the collection plus up to 120 matching editions |
| Edition | `/foundry/<collection>/edition-<n>.html` | 3,612 | Art, tier, rank, score, narrative, trait table, fingerprint, metadata link, prev/next |
| Token hub | `/tokens/index.html` | 1 | Categories, letters, first 24 concepts |
| Token directory | `/tokens/page-<n>.html` | 25 | 120 concepts per page |
| Token concept | `/tokens/c<0001>.html` | 3,000 | Concept ID, draft spec ID, category, draft standard, feature list, related concepts |
| Token category | `/tokens/category/<slug>.html` | 12 | Up to 400 concepts each |
| Token letter | `/tokens/letter/<a>.html` | 14 | Compact A to Z directory (only letters that start a name) |
| Search | `/search.html` | 1 | Live client-side search over `search-index.json` |
| Sitemap, 404 | `/sitemap.html`, `/404.html` | 2 | Hub listings |

Supporting files: `assets/style.css`, `assets/logo.jpg`, `assets/svg/<collection>/<n>.svg` (one per edition), `assets/meta/<collection>/<n>.json` (ERC-721-style metadata), `search-index.json`, `manus-routes.json`, `robots.txt`, `stats.json`.

---

## 4. The generated content

### 4.1 NFT editions

Each collection is a theme definition: ordered layers, each with weighted options, SVG fragments, and optional exclusion rules. The generator calls `createEdition(themeName, theme, edition, exclusions, used)` for every edition number. That function seeds a random generator from a SHA-256 hash of `theme:edition:attempt`, picks one option per layer by weight, rejects combinations that match an exclusion rule or reuse a fingerprint, and retries up to 2000 times.

After all editions exist, the build measures how often each trait appeared, scores every edition as the sum of `-ln(frequency)` over its traits, ranks editions from rarest, and assigns a tier by percentile: Mythic (top 1%), Legendary (5%), Epic (15%), Rare (40%), Common (the rest).

| Collection | Layers | Raw combinations | Editions built |
|---|---|---|---|
| Infinite Horizon Cosmic Nexus | 8 (Background, Horizon, Orbits, Glyph Ring, Power Core, Crest, Halo, Sparks) | 8 x 8 x 6 x 8 x 8 x 8 x 6 x 6 = 7,077,888 (before exclusions) | 3,000 |
| Matrix Heroes | 6 | 972 (before exclusions) | 600 |
| Mythic Titans | 4 | 16 | 12 |

The Cosmic Nexus art is built from code: a gradient background with deterministic stars, a curved horizon line, thin orbit ellipses, a dashed glyph ring, a radial-gradient energy core, a Greek-letter crest in gold (Omega, Xi, Sigma, Phi, Psi, Delta, Pi, Theta), an optional halo ellipse, and scattered sparks. Edition pages show the art as an `<img>` pointing at its own SVG file, which avoids duplicate SVG gradient IDs that would clash if many were inlined on one page.

### 4.2 Token concepts

Names come from 24 prefixes x 20 cores x 16 suffixes (7,680 combinations), shuffled with a fixed seed so every build is identical. If more than 7,680 are requested, a Roman numeral series suffix is added. Tickers are derived from the name and made unique. Category (12 options), draft standard (mostly ERC-20), and 3 to 5 draft features are chosen from a hash of the name.

Every token page states clearly that it is a concept: not deployed, not listed, no contract address. The "Concept ID" (`TWE-ICN-000001`) and "Draft specification ID" (`TWE-SPEC-XXXXXXXX`) are labels derived from hashes, not blockchain identifiers.

### 4.3 Safety and honesty design

* A sandbox banner appears on every page.
* The production-readiness page is a planning list for counsel, not a go-live guide.
* No credentials from the uploaded files are used or included.
* Names carry a clearance warning on the disclaimer page.

---

## 5. Search

`search-index.json` holds one array per entry: `[type, title, subtitle, url]`. The search page loads it with `fetch`, filters as you type (minimum 2 characters, 150 results maximum), and links to results using the page's `data-root` prefix. Search needs an HTTP server, so it will not work when pages are opened directly from a folder.

---

## 6. Build, hosting and publishing

* Build: `cd generator && OUT_DIR=../dist node build.mjs`. Takes about 2 seconds for 6,850 pages.
* Manus project config: `build = { command: "true", outputDirectory: "dist" }`. This publishes the committed `dist/` folder without running a build on the host.
* Preview: the platform runs `python3 -m http.server 3000 --directory dist`.
* `app.config.ts` holds the project logo URL.
* `manus-routes.json` lists the page routes (patterns for edition, gallery, trait and token pages) for the platform.
* `robots.txt` and `sitemap.xml` are handled by the platform when published.

Publishing note: the first full-size publish (about 14,000 files, 91 MB) failed twice with `builder_unavailable`. A reduced version (875 pages, about 1,700 files) published successfully in about two minutes at https://twenft-cbbwpe3q.manus.space, which points to file count or total size as the limit. The full 6,850 pages can be restored by publishing in steps.

---

## 7. Repository layout

```
twenft/
  app.config.ts          project logo metadata
  TODO.md                outcome checklist
  generator/
    build.mjs            the site generator (all templates and CSS)
    content.mjs          hand-written documentation pages
    package.json         {"type":"module"}
    vendor/
      foundry.js         the real foundry engine
      layers.json        Matrix Heroes and Mythic Titans themes
    static/
      logo.jpg           logo copied to assets/
  dist/                  generated site (committed, published)
```

---

# Source code

The complete files follow, verbatim.


## Code 1: `generator/build.mjs`

Site generator: layout template, CSS, collections, tokens, search, sitemap, home. 448 lines.

```js
// Static site generator for TWE + Standard Super NFT Foundry.
// Usage: node build.mjs      (see env knobs below)
//   COSMIC_EDITIONS=3000 MATRIX_EDITIONS=600 TOKENS=3000 SITE_URL=https://example.com node build.mjs
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';
import { createEdition, renderSvg, narrative, hash } from './vendor/foundry.js';
import { CORE, SANDBOX_TEXT } from './content.mjs';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const DIST = process.env.OUT_DIR || path.join(__dirname, 'dist');
const N_COSMIC = Number(process.env.COSMIC_EDITIONS || 3000);
const N_MATRIX = Number(process.env.MATRIX_EDITIONS || 600);
const N_TITANS = Number(process.env.TITAN_EDITIONS || 12);
const N_TOKENS = Number(process.env.TOKENS || 3000);
const SITE_URL = (process.env.SITE_URL || '').replace(/\/$/, '');
const PER_PAGE = 60;
const CREATOR = 'Mauro Lomibao Dar III';

fs.rmSync(DIST, { recursive: true, force: true });
fs.mkdirSync(DIST, { recursive: true });

/* ------------------------------ helpers ------------------------------ */
const esc = s => String(s).replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
const slug = s => String(s).toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g, '');
const pad = (n, w = 4) => String(n).padStart(w, '0');
const pages = []; // {path,title,section}
const searchIndex = [];

function writeFile(rel, data) {
  const f = path.join(DIST, rel);
  fs.mkdirSync(path.dirname(f), { recursive: true });
  fs.writeFileSync(f, data);
}
function mulberry(seed) {
  let n = seed >>> 0;
  return () => { n += 0x6D2B79F5; let t = n; t = Math.imul(t ^ t >>> 15, t | 1); t ^= t + Math.imul(t ^ t >>> 7, t | 61); return ((t ^ t >>> 14) >>> 0) / 4294967296; };
}
const seedOf = s => Number.parseInt(hash(String(s)).slice(0, 8), 16);
const roman = n => { const m = [[10, 'X'], [9, 'IX'], [5, 'V'], [4, 'IV'], [1, 'I']]; let r = ''; for (const [v, s] of m) while (n >= v) { r += s; n -= v; } return r; };
const fmt = n => Number(n).toLocaleString('en-US');

/* ------------------------------ layout ------------------------------ */
const NAV = [
  ['Home', '/index.html'], ['TWE', '/twe/index.html'], ['Foundry', '/foundry/index.html'],
  ['Collections', '/foundry/collections.html'], ['Tokens', '/tokens/index.html'], ['Search', '/search.html'], ['FAQ', '/faq.html'],
];

function layout({ p, title, desc = '', body, crumbs = [], section = '', index = true }) {
  const depth = p.split('/').length - 1;
  const pre = depth ? '../'.repeat(depth) : './';
  const crumbHtml = crumbs.length
    ? `<nav class="crumbs" aria-label="Breadcrumb">${crumbs.map(([t, u]) => u ? `<a href="${u}">${esc(t)}</a>` : `<span>${esc(t)}</span>`).join(' <i>/</i> ')}</nav>` : '';
  let html = `<!doctype html>
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>${esc(title)} | TWE &amp; NFT Foundry</title>
<meta name="description" content="${esc(desc || title)}">
<meta name="theme-color" content="#07131f">
<link rel="icon" href="/assets/logo.jpg">
<link rel="stylesheet" href="/assets/style.css">
</head><body data-root="${pre}">
<div class="banner">Non-production sandbox. ${esc(SANDBOX_TEXT)}</div>
<header class="top"><a class="brand" href="/index.html"><img src="/assets/logo.jpg" alt="" width="34" height="34"><span>TWE <b>&amp;</b> NFT Foundry</span></a>
<nav class="main">${NAV.map(([t, u]) => `<a href="${u}">${t}</a>`).join('')}</nav></header>
<main>${crumbHtml}${body}</main>
<footer><div class="cols">
<div><h4>TWE</h4><a href="/twe/index.html">Overview</a><a href="/twe/sandbox-boundary.html">Sandbox boundary</a><a href="/twe/paper-trading.html">Paper trading</a><a href="/twe/production-readiness.html">Production readiness</a></div>
<div><h4>Foundry</h4><a href="/foundry/index.html">Overview</a><a href="/foundry/how-it-works.html">How it works</a><a href="/foundry/api.html">API</a><a href="/foundry/compliance-gate.html">Compliance gate</a></div>
<div><h4>Browse</h4><a href="/foundry/collections.html">Collections</a><a href="/tokens/index.html">Token concepts</a><a href="/sitemap.html">Sitemap</a><a href="/search.html">Search</a></div>
<div><h4>About</h4><a href="/about.html">About</a><a href="/faq.html">FAQ</a><a href="/glossary.html">Glossary</a><a href="/legal/disclaimer.html">Disclaimer</a></div>
</div><p class="fine">Creator field: ${CREATOR}. Generated concepts are creative sandbox material, not offers, listings or advice.</p></footer>
</body></html>`;
  html = html.replace(/(href|src)="\/(?!\/)/g, `$1="${pre}`);
  writeFile(p, html);
  if (index) pages.push({ path: p, title, section });
}

function grid(items) { return `<div class="grid">${items.join('')}</div>`; }
function pager(base, k, total, label = 'Page') {
  if (total <= 1) return '';
  const link = (i, t) => `<a href="${base}${i}.html" ${i === k ? 'class="cur"' : ''}>${t}</a>`;
  const out = [];
  if (k > 1) out.push(link(k - 1, '&larr; Prev'));
  const window = new Set([1, 2, total - 1, total, k - 2, k - 1, k, k + 1, k + 2]);
  let last = 0;
  for (let i = 1; i <= total; i++) {
    if (!window.has(i)) continue;
    if (i - last > 1) out.push('<span>&hellip;</span>');
    out.push(link(i, i)); last = i;
  }
  if (k < total) out.push(link(k + 1, 'Next &rarr;'));
  return `<nav class="pager" aria-label="${label}">${out.join('')}</nav>`;
}

/* ------------------------------ static assets ------------------------------ */
fs.cpSync(path.join(__dirname, 'static'), path.join(DIST, 'assets'), { recursive: true });
writeFile('assets/style.css', `
:root{--bg:#07131f;--panel:#0d1f30;--panel2:#10283d;--line:#1d3a52;--text:#e6f1f8;--mute:#8fa9bc;--cy:#20d9ff;--gn:#36d7a1;--co:#ff7184;--gold:#e8c26a}
*{box-sizing:border-box}html{scroll-behavior:smooth}
body{margin:0;background:var(--bg);color:var(--text);font:16px/1.65 "Space Grotesk",system-ui,-apple-system,Segoe UI,Roboto,sans-serif}
a{color:var(--cy);text-decoration:none}a:hover{text-decoration:underline}
code,pre,.mono{font-family:"IBM Plex Mono",ui-monospace,Menlo,Consolas,monospace}
code{background:var(--panel2);padding:.1em .4em;border-radius:4px;font-size:.88em}
pre{background:var(--panel);border:1px solid var(--line);padding:14px;border-radius:8px;overflow:auto;font-size:.88em}
.banner{background:#0b2a22;color:#9be8cc;font-size:.78rem;padding:6px 14px;text-align:center;border-bottom:1px solid #14473a}
.top{display:flex;flex-wrap:wrap;align-items:center;gap:10px 22px;padding:12px 18px;border-bottom:1px solid var(--line);background:var(--panel);position:sticky;top:0;z-index:5}
.brand{display:flex;align-items:center;gap:10px;color:var(--text);font-weight:700}.brand img{border-radius:8px}.brand b{color:var(--cy)}
.main{display:flex;flex-wrap:wrap;gap:4px 16px}.main a{color:var(--mute);font-size:.95rem}.main a:hover{color:var(--cy);text-decoration:none}
main{max-width:1100px;margin:0 auto;padding:22px 16px 60px}
h1{font-size:clamp(1.7rem,4vw,2.5rem);line-height:1.15;margin:.3em 0 .2em}
h2{font-size:1.3rem;margin:1.8em 0 .4em;border-bottom:1px solid var(--line);padding-bottom:.25em}
.lead{color:var(--mute);font-size:1.12rem;max-width:760px}
.crumbs{font-size:.82rem;color:var(--mute);margin-bottom:8px}.crumbs i{opacity:.5;font-style:normal}
.hero{display:grid;grid-template-columns:1fr;gap:22px;align-items:center;margin:10px 0 30px}
@media(min-width:760px){.hero{grid-template-columns:1.3fr 1fr}}
.hero img.logo{width:100%;max-width:420px;border-radius:18px;border:1px solid var(--line);justify-self:center}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:12px;margin:18px 0}
.stat{background:var(--panel);border:1px solid var(--line);border-radius:10px;padding:12px 14px}
.stat b{display:block;font:600 1.5rem "IBM Plex Mono",monospace;color:var(--cy)}.stat span{color:var(--mute);font-size:.85rem}
.cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:14px}
.card{display:block;background:var(--panel);border:1px solid var(--line);border-radius:12px;padding:14px;color:var(--text)}
.card:hover{border-color:var(--cy);text-decoration:none}.card h3{margin:.1em 0 .3em;font-size:1.05rem}.card p{margin:0;color:var(--mute);font-size:.92rem}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(140px,1fr));gap:12px;margin:14px 0}
.thumb{background:var(--panel);border:1px solid var(--line);border-radius:10px;overflow:hidden;color:var(--text);font-size:.82rem}
.thumb:hover{border-color:var(--cy);text-decoration:none}.thumb img{display:block;width:100%;aspect-ratio:1/1;background:#000}
.thumb span{display:block;padding:6px 8px}
.tok{display:block;background:var(--panel);border:1px solid var(--line);border-radius:10px;padding:10px 12px;color:var(--text)}
.tok:hover{border-color:var(--gn);text-decoration:none}.tok b{display:block}.tok small{color:var(--mute)}.tick{font-family:"IBM Plex Mono",monospace;color:var(--gn)}
.tokgrid{display:grid;grid-template-columns:repeat(auto-fill,minmax(230px,1fr));gap:10px;margin:14px 0}
.pager{display:flex;flex-wrap:wrap;gap:6px;margin:18px 0}.pager a,.pager span{padding:4px 10px;border:1px solid var(--line);border-radius:6px;font-size:.9rem;color:var(--mute)}
.pager a.cur{background:var(--cy);color:#00202a;border-color:var(--cy)}
table{border-collapse:collapse;width:100%;margin:10px 0}th,td{text-align:left;padding:7px 10px;border-bottom:1px solid var(--line);font-size:.93rem}th{color:var(--mute);font-weight:500}
.badge{display:inline-block;padding:2px 10px;border-radius:99px;font-size:.78rem;font-weight:600;border:1px solid}
.t-Mythic{color:#ff7be5;border-color:#ff7be5}.t-Legendary{color:var(--gold);border-color:var(--gold)}.t-Epic{color:#b58cff;border-color:#b58cff}.t-Rare{color:var(--cy);border-color:var(--cy)}.t-Common{color:var(--mute);border-color:var(--line)}
.edwrap{display:grid;grid-template-columns:1fr;gap:22px}@media(min-width:800px){.edwrap{grid-template-columns:1fr 1fr}}
.edwrap img{width:100%;border-radius:14px;border:1px solid var(--line)}
.notice{background:#2a1a22;border:1px solid #5a2b3a;color:#ffc2cd;border-radius:10px;padding:10px 14px;font-size:.9rem;margin:16px 0}
.bar{height:8px;background:var(--panel2);border-radius:6px;overflow:hidden;min-width:80px}.bar i{display:block;height:100%;background:var(--cy)}
input.s{width:100%;padding:12px 14px;border-radius:10px;border:1px solid var(--line);background:var(--panel);color:var(--text);font-size:1rem}
footer{border-top:1px solid var(--line);background:var(--panel);padding:24px 16px}.cols{max-width:1100px;margin:0 auto;display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:18px}
footer h4{margin:0 0 6px;color:var(--mute);font-size:.8rem;text-transform:uppercase;letter-spacing:.08em}footer a{display:block;color:var(--text);font-size:.92rem;padding:2px 0}
.fine{max-width:1100px;margin:18px auto 0;color:var(--mute);font-size:.8rem}
.dir{columns:2 260px;column-gap:24px;font-size:.92rem}.dir a{display:block;padding:1px 0}
`);

/* ------------------------------ collections ------------------------------ */
const srcLayers = JSON.parse(fs.readFileSync(path.join(__dirname, 'vendor/layers.json'), 'utf8')).themes;

function stars(i, n = 26) {
  const r = mulberry(1000 + i); let s = '';
  for (let k = 0; k < n; k++) s += `<circle cx='${Math.floor(r() * 600)}' cy='${Math.floor(r() * 560)}' r='${(r() * 1.6 + .4).toFixed(1)}' fill='#fff' opacity='${(r() * .6 + .2).toFixed(2)}'/>`;
  return s;
}
const W = [30, 25, 18, 12, 8, 4, 2, 1];
const opt = (name, svg, weight) => ({ name, svg, weight });
const bgData = [['Event Horizon', '#05060f', '#1b1040'], ['Aurora Veil', '#04201f', '#15506a'], ['Quasar Dusk', '#1a0b2e', '#6b2d5c'], ['Nebula Bloom', '#2b0f3a', '#0b2e5c'], ['Void Silk', '#020208', '#0d0d1f'], ['Solar Tide', '#3a1204', '#c0562a'], ['Starfall Teal', '#021b24', '#0a6b6b'], ['Crimson Drift', '#220510', '#7a1230']];
const horData = [['Gold Line', '#e8c26a'], ['Cyan Rim', '#20d9ff'], ['Emerald Arc', '#36d7a1'], ['Violet Edge', '#b58cff'], ['Rose Curve', '#ff7be5'], ['Ember Line', '#ff8a3d'], ['Ice Span', '#bfe9ff'], ['White Dwarf', '#ffffff']];
const coreData = [['Amber Seed', '#ffd27a', '#b8651b'], ['Cyan Spark', '#9ff3ff', '#0a7f9e'], ['Jade Pulse', '#9fffd9', '#0e8a63'], ['Violet Ember', '#e3c9ff', '#6b2fb5'], ['Rose Quartz', '#ffc9ee', '#b03a8f'], ['Solar Flare', '#fff1a8', '#e8552b'], ['Silver Star', '#ffffff', '#7a8aa0'], ['Void Pearl', '#2a2a4a', '#000000']];
const ringData = [['Fine Dash', '2 8', 2], ['Long Dash', '24 12', 3], ['Dotted', '1 9', 5], ['Solid Thin', '0', 1.5], ['Solid Bold', '0', 6], ['Broken', '90 40', 4], ['Double', '0', 2], ['Pulse', '4 4 20 4', 3]];
const crestData = [['Omega Seal', 'Ω'], ['Xi Seal', 'Ξ'], ['Sigma Seal', 'Σ'], ['Phi Seal', 'Φ'], ['Psi Seal', 'Ψ'], ['Delta Seal', 'Δ'], ['Pi Seal', 'Π'], ['Theta Seal', 'Θ']];
const haloData = [['No Halo', null], ['Thin Halo', 'a'], ['Gold Halo', 'b'], ['Twin Halo', 'c'], ['Dashed Halo', 'd'], ['Wide Halo', 'e']];
const sparkData = [['No Sparks', 0], ['Faint Sparks', 8], ['Sparks', 16], ['Bright Sparks', 28], ['Storm', 48], ['Supernova', 80]];
const orbitData = [['No Orbits', 0], ['Single Orbit', 1], ['Twin Orbits', 2], ['Triple Orbits', 3], ['Cross Orbits', 4], ['Gyroscope', 5]];

const cosmicDef = {
  description: 'a boundless horizon where liquidity, light and connection never end.',
  exclusions: [['Background:Event Horizon', 'Power Core:Void Pearl'], ['Background:Void Silk', 'Power Core:Void Pearl'], ['Horizon:Ember Line', 'Background:Crimson Drift']],
  layers: [
    { trait_type: 'Background', options: bgData.map(([n, a, b], i) => opt(n, `<defs><linearGradient id='bg' x1='0' y1='0' x2='.6' y2='1'><stop stop-color='${a}'/><stop offset='1' stop-color='${b}'/></linearGradient></defs><rect width='600' height='600' fill='url(#bg)'/>${stars(i)}`, W[i])) },
    { trait_type: 'Horizon', options: horData.map(([n, c], i) => opt(n, `<path d='M0 400 Q300 ${320 + i * 7} 600 400 V600 H0Z' fill='${c}' opacity='.14'/><path d='M0 400 Q300 ${320 + i * 7} 600 400' stroke='${c}' stroke-width='${2 + (i % 3)}' fill='none'/><path d='M0 420 Q300 ${350 + i * 7} 600 420' stroke='${c}' stroke-width='1' fill='none' opacity='.5'/>`, W[i])) },
    { trait_type: 'Orbits', options: orbitData.map(([n, k], i) => opt(n, Array.from({ length: k }, (_, j) => `<ellipse cx='300' cy='290' rx='${230 - j * 18}' ry='${70 + j * 14}' transform='rotate(${j * (180 / Math.max(k, 1)) + 15} 300 290)' fill='none' stroke='#fff' stroke-opacity='.28' stroke-width='1.5'/>`).join(''), [30, 25, 18, 14, 8, 5][i])) },
    { trait_type: 'Glyph Ring', options: ringData.map(([n, d, w], i) => opt(n, `<circle cx='300' cy='290' r='150' fill='none' stroke='#fff' stroke-opacity='.55' stroke-width='${w}' stroke-dasharray='${d}'/>${n === 'Double' ? "<circle cx='300' cy='290' r='164' fill='none' stroke='#fff' stroke-opacity='.4' stroke-width='1.5'/>" : ''}`, W[i])) },
    { trait_type: 'Power Core', options: coreData.map(([n, a, b], i) => opt(n, `<defs><radialGradient id='core'><stop stop-color='${a}'/><stop offset='1' stop-color='${b}'/></radialGradient></defs><circle cx='300' cy='290' r='${74 - (i % 3) * 4}' fill='url(#core)'/><circle cx='300' cy='290' r='${100 - (i % 3) * 4}' fill='none' stroke='${a}' stroke-opacity='.35' stroke-width='3'/>`, W[i])) },
    { trait_type: 'Crest', options: crestData.map(([n, g], i) => opt(n, `<text x='300' y='480' text-anchor='middle' font-family='Georgia,serif' font-size='${56 + (i % 3) * 4}' fill='#e8c26a' stroke='#3a2a05' stroke-width='1'>${g}</text>`, W[i])) },
    { trait_type: 'Halo', options: haloData.map(([n, k], i) => opt(n, ({ null: '', a: "<ellipse cx='300' cy='96' rx='90' ry='22' fill='none' stroke='#fff' stroke-width='3' opacity='.8'/>", b: "<ellipse cx='300' cy='96' rx='96' ry='24' fill='none' stroke='#e8c26a' stroke-width='7'/>", c: "<ellipse cx='300' cy='96' rx='90' ry='22' fill='none' stroke='#e8c26a' stroke-width='4'/><ellipse cx='300' cy='84' rx='70' ry='16' fill='none' stroke='#e8c26a' stroke-width='3'/>", d: "<ellipse cx='300' cy='96' rx='100' ry='26' fill='none' stroke='#20d9ff' stroke-width='4' stroke-dasharray='10 8'/>", e: "<ellipse cx='300' cy='90' rx='150' ry='34' fill='none' stroke='#ff7be5' stroke-width='3' opacity='.85'/>" })[k] ?? '', [30, 25, 20, 12, 8, 5][i])) },
    { trait_type: 'Sparks', options: sparkData.map(([n, k], i) => opt(n, (() => { const r = mulberry(77 + i); let s = ''; for (let j = 0; j < k; j++) s += `<circle cx='${Math.floor(r() * 600)}' cy='${Math.floor(r() * 520)}' r='${(r() * 2.4 + .8).toFixed(1)}' fill='#ffd27a' opacity='${(r() * .6 + .3).toFixed(2)}'/>`; return s; })(), [30, 25, 20, 14, 8, 3][i])) },
  ],
};

const COLLECTIONS = [
  { slug: 'cosmic-nexus', name: 'Infinite Horizon Cosmic Nexus', def: cosmicDef, n: N_COSMIC, blurb: 'A new 8-layer collection built for this site, with millions of possible combinations.' },
  { slug: 'matrix-heroes', name: 'Matrix Heroes', def: srcLayers['Matrix Heroes'], n: N_MATRIX, blurb: 'Digital guardians born from the code stream (theme from the foundry config).' },
  { slug: 'mythic-titans', name: 'Mythic Titans', def: srcLayers['Mythic Titans'], n: N_TITANS, blurb: 'Fire and ice champions. Only 16 combinations exist, so the engine can mint at most 16 unique editions.' },
];

const tierOf = pct => pct <= .01 ? 'Mythic' : pct <= .05 ? 'Legendary' : pct <= .15 ? 'Epic' : pct <= .40 ? 'Rare' : 'Common';

for (const col of COLLECTIONS) {
  console.log('Generating', col.name);
  const used = new Set(); const eds = [];
  for (let i = 1; i <= col.n; i++) {
    try {
      const { traits, fingerprint } = createEdition(col.name, col.def, i, col.def.exclusions || [], used);
      eds.push({ i, traits, fingerprint });
    } catch (e) { console.log('  stopped at', i - 1, '-', e.message.slice(0, 60)); break; }
  }
  col.eds = eds;
  const n = eds.length;
  const freq = {};
  for (const e of eds) for (const t of e.traits) { const k = `${t.trait_type}:${t.option.name}`; freq[k] = (freq[k] || 0) + 1; }
  col.freq = freq;
  for (const e of eds) e.score = e.traits.reduce((s, t) => s + -Math.log(freq[`${t.trait_type}:${t.option.name}`] / n), 0);
  const ranked = [...eds].sort((a, b) => b.score - a.score);
  ranked.forEach((e, r) => { e.rank = r + 1; e.tier = tierOf((r + 1) / n); });
  for (const e of eds) {
    e.title = `${col.name} #${e.i}`;
    e.story = narrative(col.def, e.traits, e.i);
    writeFile(`assets/svg/${col.slug}/${e.i}.svg`, renderSvg(e.traits, e.title, e.i));
    writeFile(`assets/meta/${col.slug}/${e.i}.json`, JSON.stringify({
      name: e.title, description: e.story, image: `../../svg/${col.slug}/${e.i}.svg`, edition: e.i,
      attributes: e.traits.map(t => ({ trait_type: t.trait_type, value: t.option.name })),
      properties: { fingerprint: e.fingerprint, theme: col.name, creator: CREATOR, rarity_rank: e.rank, rarity_tier: e.tier },
    }, null, 2));
    searchIndex.push(['Edition', e.title, e.tier, `foundry/${col.slug}/edition-${e.i}.html`]);
  }
  col.pageCount = Math.max(1, Math.ceil(n / PER_PAGE));
  const thumb = e => `<a class="thumb" href="/foundry/${col.slug}/edition-${e.i}.html"><img loading="lazy" src="/assets/svg/${col.slug}/${e.i}.svg" alt="${esc(e.title)}"><span>#${e.i} <em class="badge t-${e.tier}">${e.tier}</em></span></a>`;
  const traitUrl = (t) => `/foundry/traits/${col.slug}/${slug(t.trait_type)}--${slug(t.option.name)}.html`;

  // collection overview
  const layerRows = col.def.layers.map(l => `<tr><td>${esc(l.trait_type)}</td><td>${l.options.map(o => `<a href="/foundry/traits/${col.slug}/${slug(l.trait_type)}--${slug(o.name)}.html">${esc(o.name)}</a>`).join(', ')}</td></tr>`).join('');
  const space = col.def.layers.reduce((a, l) => a * l.options.length, 1);
  layout({
    p: `foundry/${col.slug}/index.html`, section: 'Collection', title: col.name,
    desc: `${col.name}: ${fmt(n)} generated editions with unique SVG art.`,
    crumbs: [['Home', '/index.html'], ['Collections', '/foundry/collections.html'], [col.name]],
    body: `<h1>${esc(col.name)}</h1><p class="lead">${esc(col.blurb)}</p>
<div class="stats"><div class="stat"><b>${fmt(n)}</b><span>editions generated</span></div><div class="stat"><b>${fmt(space)}</b><span>raw trait combinations</span></div><div class="stat"><b>${col.def.layers.length}</b><span>layers</span></div><div class="stat"><b>${col.pageCount}</b><span>gallery pages</span></div></div>
<h2>Layers and options</h2><table><tr><th>Layer</th><th>Options</th></tr>${layerRows}</table>
${(col.def.exclusions || []).length ? `<h2>Exclusion rules</h2><ul>${col.def.exclusions.map(r => `<li><code>${esc(r.join(' + '))}</code></li>`).join('')}</ul>` : ''}
<h2>Sample editions</h2>${grid(eds.slice(0, 12).map(thumb))}
<p><a href="/foundry/${col.slug}/page-1.html">Browse all ${fmt(n)} editions &rarr;</a> &middot; <a href="/foundry/${col.slug}/rarity.html">Rarity table</a></p>`,
  });
  // gallery pages
  for (let k = 1; k <= col.pageCount; k++) {
    const slice = eds.slice((k - 1) * PER_PAGE, k * PER_PAGE);
    layout({
      p: `foundry/${col.slug}/page-${k}.html`, section: 'Gallery', title: `${col.name} gallery, page ${k}`,
      desc: `Editions ${slice[0]?.i}-${slice.at(-1)?.i} of ${col.name}.`,
      crumbs: [['Home', '/index.html'], ['Collections', '/foundry/collections.html'], [col.name, `/foundry/${col.slug}/index.html`], [`Page ${k}`]],
      body: `<h1>${esc(col.name)} - page ${k} of ${col.pageCount}</h1>${grid(slice.map(thumb))}${pager(`/foundry/${col.slug}/page-`, k, col.pageCount)}`,
    });
  }
  // rarity table
  const rrows = col.def.layers.flatMap(l => l.options.map(o => { const c = freq[`${l.trait_type}:${o.name}`] || 0; return { l: l.trait_type, o: o.name, c, pct: n ? c / n * 100 : 0 }; }));
  layout({
    p: `foundry/${col.slug}/rarity.html`, section: 'Collection', title: `${col.name} rarity table`,
    crumbs: [['Home', '/index.html'], ['Collections', '/foundry/collections.html'], [col.name, `/foundry/${col.slug}/index.html`], ['Rarity']],
    body: `<h1>${esc(col.name)} rarity</h1><p class="lead">Measured over all ${fmt(n)} generated editions.</p><table><tr><th>Layer</th><th>Trait</th><th>Count</th><th>Share</th><th></th></tr>${rrows.map(r => `<tr><td>${esc(r.l)}</td><td><a href="/foundry/traits/${col.slug}/${slug(r.l)}--${slug(r.o)}.html">${esc(r.o)}</a></td><td class="mono">${fmt(r.c)}</td><td class="mono">${r.pct.toFixed(2)}%</td><td><div class="bar"><i style="width:${Math.min(100, r.pct * 2.2).toFixed(0)}%"></i></div></td></tr>`).join('')}</table>`,
  });
  // trait pages
  for (const l of col.def.layers) for (const o of l.options) {
    const list = eds.filter(e => e.traits.some(t => t.trait_type === l.trait_type && t.option.name === o.name));
    const c = list.length;
    searchIndex.push(['Trait', `${o.name} (${l.trait_type})`, col.name, `foundry/traits/${col.slug}/${slug(l.trait_type)}--${slug(o.name)}.html`]);
    layout({
      p: `foundry/traits/${col.slug}/${slug(l.trait_type)}--${slug(o.name)}.html`, section: 'Trait', title: `${o.name} (${l.trait_type}) in ${col.name}`,
      crumbs: [['Home', '/index.html'], [col.name, `/foundry/${col.slug}/index.html`], ['Traits'], [`${l.trait_type}: ${o.name}`]],
      body: `<h1>${esc(o.name)}</h1><p class="lead">${esc(l.trait_type)} trait in ${esc(col.name)}. Appears in <b>${fmt(c)}</b> of ${fmt(n)} editions (${n ? (c / n * 100).toFixed(2) : 0}%). Configured weight: ${o.weight}.</p>${c ? grid(list.slice(0, 120).map(thumb)) : '<p>No editions carry this trait in the generated set.</p>'}${c > 120 ? `<p>Showing the first 120 of ${fmt(c)}.</p>` : ''}`,
    });
  }
  // edition pages
  for (let x = 0; x < eds.length; x++) {
    const e = eds[x]; const prev = eds[x - 1], next = eds[x + 1];
    const rows = e.traits.map(t => { const c = freq[`${t.trait_type}:${t.option.name}`]; return `<tr><td>${esc(t.trait_type)}</td><td><a href="${traitUrl(t)}">${esc(t.option.name)}</a></td><td class="mono">${(c / n * 100).toFixed(2)}%</td></tr>`; }).join('');
    layout({
      p: `foundry/${col.slug}/edition-${e.i}.html`, section: 'Edition', title: e.title, desc: e.story.slice(0, 150),
      crumbs: [['Home', '/index.html'], ['Collections', '/foundry/collections.html'], [col.name, `/foundry/${col.slug}/index.html`], [`#${e.i}`]],
      body: `<div class="edwrap"><div><img src="/assets/svg/${col.slug}/${e.i}.svg" alt="${esc(e.title)}"></div><div>
<h1>${esc(e.title)}</h1><p><span class="badge t-${e.tier}">${e.tier}</span> &nbsp; Rarity rank <b class="mono">${e.rank}</b> of ${fmt(n)} &middot; score <span class="mono">${e.score.toFixed(2)}</span></p>
<p>${esc(e.story)}</p><table><tr><th>Layer</th><th>Trait</th><th>Share</th></tr>${rows}</table>
<p class="mono" style="word-break:break-all;color:var(--mute);font-size:.8rem">fingerprint ${e.fingerprint}</p>
<p><a href="/assets/meta/${col.slug}/${e.i}.json">Metadata JSON</a> &middot; <a href="/assets/svg/${col.slug}/${e.i}.svg">SVG file</a></p>
<p>${prev ? `<a href="/foundry/${col.slug}/edition-${prev.i}.html">&larr; #${prev.i}</a>` : ''} ${next ? ` &nbsp; <a href="/foundry/${col.slug}/edition-${next.i}.html">#${next.i} &rarr;</a>` : ''}</p>
</div></div>`,
    });
  }
}

layout({
  p: 'foundry/collections.html', section: 'Foundry', title: 'Collections',
  crumbs: [['Home', '/index.html'], ['Collections']],
  body: `<h1>Collections</h1><p class="lead">Every edition here was produced by the foundry engine and has its own page, SVG and metadata file.</p><div class="cards">${COLLECTIONS.map(c => `<a class="card" href="/foundry/${c.slug}/index.html"><h3>${esc(c.name)}</h3><p>${fmt(c.eds.length)} editions. ${esc(c.blurb)}</p></a>`).join('')}</div>
<h2>Sample art</h2>${grid(COLLECTIONS[0].eds.slice(0, 18).map(e => `<a class="thumb" href="/foundry/cosmic-nexus/edition-${e.i}.html"><img loading="lazy" src="/assets/svg/cosmic-nexus/${e.i}.svg" alt=""><span>#${e.i}</span></a>`))}`,
});

/* ------------------------------ token concepts ------------------------------ */
console.log('Generating token concepts');
const PRE = ['Aether', 'Astra', 'Nova', 'Zenith', 'Umbra', 'Lumen', 'Vertex', 'Helix', 'Cosmo', 'Halo', 'Quanta', 'Stellar', 'Ion', 'Solar', 'Lunar', 'Photon', 'Nebula', 'Pulsar', 'Aurora', 'Vega', 'Titan', 'Eon', 'Apex', 'Cirrus'];
const COR = ['Horizon', 'Nexus', 'Gate', 'Beacon', 'Bridge', 'Vault', 'Signal', 'Orbit', 'Rift', 'Span', 'Loom', 'Forge', 'Current', 'Lattice', 'Meridian', 'Anchor', 'Prism', 'Arc', 'Tide', 'Reach'];
const SUF = ['Protocol', 'Reserve', 'Index', 'Credit', 'Works', 'Labs', 'Network', 'Commons', 'Unit', 'Pass', 'Ledger', 'Core', 'Dynamics', 'Collective', 'Standard', 'Relay'];
const CATS = [
  ['Liquidity Infrastructure', 'shared liquidity routing across venues and pools'],
  ['Settlement Rails', 'fast, auditable settlement between counterparties'],
  ['Cross-Chain Connectivity', 'message and asset passing between independent networks'],
  ['Data and Oracles', 'verifiable data feeds for contracts and applications'],
  ['Identity and Access', 'portable credentials and permissioned access'],
  ['Governance', 'transparent proposal, voting and treasury workflows'],
  ['Resource Credits', 'metering of compute, bandwidth or carbon accounting units'],
  ['Creative Rights', 'licensing and royalty tracking for creative works'],
  ['Gaming and Virtual Worlds', 'in-world currencies and item economies'],
  ['Supply-Chain Provenance', 'tamper-evident tracking from source to shelf'],
  ['Risk and Insurance', 'pooled risk coverage and parametric triggers'],
  ['Education and Research', 'credentialing and open-research funding'],
];
const FEATURES = ['Role-based minting', 'Pausable transfers', 'Snapshot-based voting', 'Allowlist gating', 'Time-locked vesting', 'Capped supply', 'Burn-on-redeem', 'Permit signatures', 'Upgrade timelock', 'Per-wallet transfer limits'];
const STANDARDS = ['ERC-20 (fungible)', 'ERC-20 (fungible)', 'ERC-20 (fungible)', 'ERC-20 (fungible)', 'ERC-1155 (multi-asset)', 'ERC-721 (non-fungible)'];
const combos = [];
for (const p of PRE) for (const c of COR) for (const s of SUF) combos.push([p, c, s]);
{ const r = mulberry(20261123); for (let i = combos.length - 1; i > 0; i--) { const j = Math.floor(r() * (i + 1)); [combos[i], combos[j]] = [combos[j], combos[i]]; } }
const tickers = new Set(); const tokens = [];
for (let i = 0; i < N_TOKENS; i++) {
  const [p, c, s] = combos[i % combos.length]; const round = Math.floor(i / combos.length);
  const name = `${p} ${c} ${s}${round ? ' ' + roman(round + 1) : ''}`;
  let t = (p.slice(0, 2) + c[0] + s[0]).toUpperCase();
  if (tickers.has(t)) t = (p.slice(0, 3) + c[0] + s[0]).toUpperCase();
  let h = hash(name), k = 0; while (tickers.has(t)) { t = (p.slice(0, 2) + c[0] + h[k++]).toUpperCase() + (k > 6 ? h[k] : ''); }
  tickers.add(t);
  const hs = hash(`tok:${name}`); const hv = parseInt(hs.slice(0, 8), 16);
  const cat = CATS[hv % CATS.length];
  const r = mulberry(hv);
  const feats = [...FEATURES].sort(() => r() - .5).slice(0, 3 + Math.floor(r() * 3));
  tokens.push({
    i: i + 1, id: pad(i + 1), name, ticker: t, cat, slug: slug(cat[0]), feats, std: STANDARDS[hv % STANDARDS.length],
    spec: 'TWE-SPEC-' + hs.slice(8, 16).toUpperCase(), cid: 'TWE-ICN-' + pad(i + 1, 6),
    blurb: `${name} (${t}) is a concept for ${cat[1]}, themed on the Infinite Horizon / Cosmic Nexus: boundless connection without a fixed edge.`,
  });
  searchIndex.push(['Token concept', name, t, `tokens/c${pad(i + 1)}.html`]);
}
const tokCard = t => `<a class="tok" href="/tokens/c${t.id}.html"><b>${esc(t.name)}</b><small><span class="tick">${t.ticker}</span> &middot; ${esc(t.cat[0])}</small></a>`;
const tokPages = Math.max(1, Math.ceil(tokens.length / 120));
for (let x = 0; x < tokens.length; x++) {
  const t = tokens[x]; const prev = tokens[x - 1], next = tokens[x + 1];
  const same = tokens.filter(o => o.slug === t.slug && o.i !== t.i);
  const rel = [1, 2, 3, 4, 5, 6].map(d => same[(x * 7 + d * 13) % Math.max(same.length, 1)]).filter(Boolean);
  layout({
    p: `tokens/c${t.id}.html`, section: 'Token concept', title: `${t.name} (${t.ticker}) concept`, desc: t.blurb,
    crumbs: [['Home', '/index.html'], ['Token concepts', '/tokens/index.html'], [t.cat[0], `/tokens/category/${t.slug}.html`], [t.name]],
    body: `<h1>${esc(t.name)} <span class="tick">${t.ticker}</span></h1><p class="lead">${esc(t.blurb)}</p>
<div class="notice"><b>Concept only.</b> Not deployed, not listed, no contract address, not an offer. The IDs below are labels derived from a hash.</div>
<table><tr><th>Concept ID</th><td class="mono">${t.cid}</td></tr><tr><th>Draft specification ID</th><td class="mono">${t.spec}</td></tr><tr><th>Category</th><td><a href="/tokens/category/${t.slug}.html">${esc(t.cat[0])}</a></td></tr><tr><th>Draft token standard</th><td>${t.std}</td></tr><tr><th>Theme</th><td>Infinite Horizon / Cosmic Nexus</td></tr><tr><th>Status</th><td>Concept (private sandbox batch)</td></tr></table>
<h2>Draft feature set</h2><ul>${t.feats.map(f => `<li>${f}</li>`).join('')}</ul>
<h2>Before this could ever be real</h2><p>A name clearance search, legal classification by qualified counsel, an independently audited contract, and a compliant distribution plan. See <a href="/twe/production-readiness.html">production-readiness planning</a>.</p>
${rel.length ? `<h2>Related concepts</h2><div class="tokgrid">${rel.map(tokCard).join('')}</div>` : ''}
<p>${prev ? `<a href="/tokens/c${prev.id}.html">&larr; ${esc(prev.name)}</a>` : ''}${next ? ` &nbsp; <a href="/tokens/c${next.id}.html">${esc(next.name)} &rarr;</a>` : ''}</p>`,
  });
}
for (let k = 1; k <= tokPages; k++) {
  const slice = tokens.slice((k - 1) * 120, k * 120);
  layout({
    p: `tokens/page-${k}.html`, section: 'Token directory', title: `Token concepts, page ${k}`,
    crumbs: [['Home', '/index.html'], ['Token concepts', '/tokens/index.html'], [`Page ${k}`]],
    body: `<h1>Token concepts - page ${k} of ${tokPages}</h1><div class="tokgrid">${slice.map(tokCard).join('')}</div>${pager('/tokens/page-', k, tokPages)}`,
  });
}
const catMap = new Map();
for (const t of tokens) { if (!catMap.has(t.slug)) catMap.set(t.slug, { name: t.cat[0], desc: t.cat[1], list: [] }); catMap.get(t.slug).list.push(t); }
for (const [s, c] of catMap) layout({
  p: `tokens/category/${s}.html`, section: 'Token category', title: `${c.name} token concepts`,
  crumbs: [['Home', '/index.html'], ['Token concepts', '/tokens/index.html'], [c.name]],
  body: `<h1>${esc(c.name)}</h1><p class="lead">Concepts for ${esc(c.desc)}. ${fmt(c.list.length)} in this batch${c.list.length > 400 ? ' (first 400 shown)' : ''}.</p><div class="tokgrid">${c.list.slice(0, 400).map(tokCard).join('')}</div>`,
});
const letters = new Map();
for (const t of tokens) { const L = t.name[0].toUpperCase(); if (!letters.has(L)) letters.set(L, []); letters.get(L).push(t); }
for (const [L, list] of letters) layout({
  p: `tokens/letter/${L.toLowerCase()}.html`, section: 'Token directory', title: `Token concepts starting with ${L}`,
  crumbs: [['Home', '/index.html'], ['Token concepts', '/tokens/index.html'], [`Letter ${L}`]],
  body: `<h1>Letter ${L}</h1><p class="lead">${fmt(list.length)} concepts.</p><div class="dir">${list.map(t => `<a href="/tokens/c${t.id}.html">${esc(t.name)} <span class="tick">${t.ticker}</span></a>`).join('')}</div>`,
});
layout({
  p: 'tokens/index.html', section: 'Token directory', title: 'Token concept directory',
  desc: `${fmt(tokens.length)} deterministic token concepts themed on Infinite Horizon / Cosmic Nexus.`,
  crumbs: [['Home', '/index.html'], ['Token concepts']],
  body: `<h1>Token concept directory</h1><p class="lead">${fmt(tokens.length)} deterministic concepts for the Infinite Horizon / Cosmic Nexus theme. Same seed, same batch.</p>
<div class="notice"><b>Concepts, not tokens.</b> Nothing is deployed or listed. Identifiers are labels, not addresses.</div>
<h2>By category</h2><div class="cards">${[...catMap].map(([s, c]) => `<a class="card" href="/tokens/category/${s}.html"><h3>${esc(c.name)}</h3><p>${fmt(c.list.length)} concepts</p></a>`).join('')}</div>
<h2>By letter</h2><div class="pager">${[...letters.keys()].sort().map(L => `<a href="/tokens/letter/${L.toLowerCase()}.html">${L}</a>`).join('')}</div>
<h2>All concepts</h2><p><a href="/tokens/page-1.html">Browse page by page (${tokPages} pages)</a></p><div class="tokgrid">${tokens.slice(0, 24).map(tokCard).join('')}</div>`,
});

/* ------------------------------ core pages ------------------------------ */
for (const pg of CORE) {
  const body = `<h1>${pg.title}</h1><p class="lead">${pg.lead}</p>` + pg.sections.map(s => `<h2>${s.h}</h2>${(s.p || []).map(x => x.startsWith('<pre') ? x : `<p>${x}</p>`).join('')}${s.ul ? `<ul>${s.ul.map(x => `<li>${x}</li>`).join('')}</ul>` : ''}`).join('');
  layout({ p: pg.path, section: pg.section, title: pg.title.replace(/<[^>]+>/g, ''), desc: pg.lead.replace(/<[^>]+>/g, ''), body, crumbs: [['Home', '/index.html'], [pg.section]] });
  searchIndex.push(['Page', pg.title.replace(/<[^>]+>/g, ''), pg.section, pg.path]);
}

/* ------------------------------ search, sitemap, 404, home ------------------------------ */
writeFile('search-index.json', JSON.stringify(searchIndex));
layout({
  p: 'search.html', section: 'Site', title: 'Search', crumbs: [['Home', '/index.html'], ['Search']],
  body: `<h1>Search</h1><p class="lead">Search ${fmt(searchIndex.length)} pages: editions, token concepts and documentation. (Needs a web server; it will not load from a file:// folder.)</p>
<input class="s" id="q" placeholder="Try: Omega, Mythic, Aether Horizon, API" autofocus><p id="n" class="lead"></p><div class="dir" id="r" style="columns:1"></div>
<script>
(async()=>{const root=document.body.dataset.root;const data=await (await fetch(root+'search-index.json')).json();
const q=document.getElementById('q'),r=document.getElementById('r'),n=document.getElementById('n');
function run(){const v=q.value.trim().toLowerCase();if(v.length<2){r.innerHTML='';n.textContent='';return}
const out=[];for(const d of data){if((d[1]+' '+d[2]).toLowerCase().includes(v)){out.push(d);if(out.length>=150)break}}
n.textContent=out.length+(out.length>=150?'+':'')+' results';
r.innerHTML=out.map(d=>'<a href="'+root+d[3]+'"><b>'+d[1].replace(/</g,'&lt;')+'</b> <small style="color:#8fa9bc">'+d[0]+' &middot; '+d[2]+'</small></a>').join('')}
q.addEventListener('input',run)})();
</script>`,
});
layout({
  p: 'sitemap.html', section: 'Site', title: 'Sitemap', crumbs: [['Home', '/index.html'], ['Sitemap']],
  body: `<h1>Sitemap</h1><h2>Documentation</h2><div class="dir">${CORE.map(c => `<a href="/${c.path}">${c.title.replace(/<[^>]+>/g, '')}</a>`).join('')}</div>
<h2>Collections</h2><div class="dir">${COLLECTIONS.flatMap(c => [`<a href="/foundry/${c.slug}/index.html">${esc(c.name)}</a>`, `<a href="/foundry/${c.slug}/page-1.html">${esc(c.name)} gallery (${c.pageCount} pages)</a>`, `<a href="/foundry/${c.slug}/rarity.html">${esc(c.name)} rarity</a>`]).join('')}</div>
<h2>Token concepts</h2><div class="dir"><a href="/tokens/index.html">Directory</a><a href="/tokens/page-1.html">All concepts (${tokPages} pages)</a>${[...catMap].map(([s, c]) => `<a href="/tokens/category/${s}.html">${esc(c.name)}</a>`).join('')}${[...letters.keys()].sort().map(L => `<a href="/tokens/letter/${L.toLowerCase()}.html">Letter ${L}</a>`).join('')}</div>`,
});
layout({ p: '404.html', section: 'Site', title: 'Not found', body: '<h1>Page not found</h1><p class="lead">Try the <a href="/sitemap.html">sitemap</a> or <a href="/search.html">search</a>.</p>' });

const totalPages = pages.length + 1; // + home
const secCount = {}; for (const p of pages) secCount[p.section] = (secCount[p.section] || 0) + 1;
const totalEditions = COLLECTIONS.reduce((a, c) => a + c.eds.length, 0);
layout({
  p: 'index.html', section: 'Home', title: 'TWE and NFT Foundry',
  desc: 'Documentation and a generated catalogue for Token World Exchange ATS (TWE) and the Standard Super NFT Foundry.',
  body: `<div class="hero"><div><h1>Token World Exchange ATS &amp; the Standard Super NFT Foundry</h1><p class="lead">A sandboxed exchange-design workspace and a deterministic generative-art engine, documented page by page, with a catalogue of ${fmt(totalEditions)} generated editions and ${fmt(tokens.length)} token concepts.</p>
<p><a class="badge t-Rare" href="/twe/index.html">Explore TWE</a> &nbsp; <a class="badge t-Legendary" href="/foundry/collections.html">Browse editions</a> &nbsp; <a class="badge t-Common" href="/tokens/index.html">Token concepts</a></p></div>
<img class="logo" src="/assets/logo.jpg" alt="MD The Gods emblem"></div>
<div class="stats"><div class="stat"><b>${fmt(totalPages)}</b><span>pages on this site</span></div><div class="stat"><b>${fmt(totalEditions)}</b><span>unique NFT editions</span></div><div class="stat"><b>${fmt(tokens.length)}</b><span>token concepts</span></div><div class="stat"><b>${COLLECTIONS.length}</b><span>collections</span></div></div>
<div class="cards"><a class="card" href="/twe/index.html"><h3>TWE overview</h3><p>Paper trading, concept generator, planner and architecture workspace.</p></a><a class="card" href="/foundry/index.html"><h3>NFT Foundry</h3><p>Weighted traits, exclusions, fingerprints, SVG, metadata.</p></a><a class="card" href="/twe/sandbox-boundary.html"><h3>Sandbox boundary</h3><p>What is deliberately not connected.</p></a><a class="card" href="/twe/production-readiness.html"><h3>Production readiness</h3><p>A planning checklist for counsel and compliance.</p></a></div>
<h2>Fresh from the foundry</h2>${grid(COLLECTIONS[0].eds.slice(0, 24).map(e => `<a class="thumb" href="/foundry/cosmic-nexus/edition-${e.i}.html"><img loading="lazy" src="/assets/svg/cosmic-nexus/${e.i}.svg" alt=""><span>#${e.i} <em class="badge t-${e.tier}">${e.tier}</em></span></a>`))}`,
});

if (SITE_URL) {
  writeFile('sitemap.xml', `<?xml version="1.0" encoding="UTF-8"?>\n<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n${pages.map(p => `<url><loc>${SITE_URL}/${p.path}</loc></url>`).join('\n')}\n</urlset>`);
  writeFile('robots.txt', `User-agent: *\nAllow: /\nSitemap: ${SITE_URL}/sitemap.xml\n`);
} else writeFile('robots.txt', 'User-agent: *\nAllow: /\n');
writeFile('manus-routes.json', JSON.stringify({ routes: [
  { path: '/', title: 'Home' },
  ...CORE.map(c => ({ path: '/' + c.path.replace(/index\.html$/, ''), title: c.title.replace(/<[^>]+>/g, '') })),
  { path: '/foundry/collections.html', title: 'Collections' },
  { path: '/tokens/', title: 'Token concept directory' },
  { path: '/search.html', title: 'Search' },
  { path: '/sitemap.html', title: 'Sitemap' },
  { path: '/foundry/:collection/', title: 'Collection overview' },
  { path: '/foundry/:collection/page-:n.html', title: 'Gallery page' },
  { path: '/foundry/:collection/edition-:n.html', title: 'NFT edition' },
  { path: '/foundry/:collection/rarity.html', title: 'Rarity table' },
  { path: '/foundry/traits/:collection/:trait.html', title: 'Trait' },
  { path: '/tokens/page-:n.html', title: 'Token concepts page' },
  { path: '/tokens/c:id.html', title: 'Token concept' },
  { path: '/tokens/category/:category.html', title: 'Token category' },
  { path: '/tokens/letter/:letter.html', title: 'Token letter directory' },
] }, null, 1));
writeFile('stats.json', JSON.stringify({ totalPages, bySection: secCount, editions: totalEditions, tokens: tokens.length, collections: COLLECTIONS.map(c => ({ name: c.name, editions: c.eds.length })) }, null, 2));
console.log('Pages:', totalPages, secCount);
```

## Code 2: `generator/content.mjs`

Hand-written documentation pages (TWE, Foundry, About, Legal). 313 lines.

```js
// Hand-written core pages for the TWE + NFT Foundry site.
// Each page: { path, title, lead, section, sections:[{h, p:[], ul:[]}] }
// All facts are drawn from the uploaded TWE README and the Standard Super NFT Foundry README/docs.

const SANDBOX =
  'No real money is accepted or moved. Orders and balances are simulated. Nothing here is a licensed exchange, an offering, or investment advice.';

export const CORE = [
  /* ---------------------------- TWE ---------------------------- */
  {
    path: 'twe/index.html', section: 'TWE', title: 'Token World Exchange ATS (TWE)',
    lead: 'Signal Terminal: a full-stack, non-production exchange-design and paper-trading sandbox.',
    sections: [
      { h: 'What TWE is', p: [
        'TWE brings a sample token market, a paper account, a deterministic token-concept generator, a private NFT collection planner and an architecture-planning workspace into one responsive application.',
        'It is a working software prototype, not a licensed exchange or an operational trading venue. ' + SANDBOX ] },
      { h: 'The five working areas', ul: [
        'Sample market and order book (illustrative data only)',
        'Paper account with a simulated USD balance and sample holdings',
        'Deterministic token-concept generator (seeded, repeatable)',
        'Private NFT collection planner',
        'Architecture-planning workspace with draft contract specifications' ] },
      { h: 'Where to go next', ul: [
        '<a href="/twe/sandbox-boundary.html">Sandbox boundary</a> - what is deliberately disconnected',
        '<a href="/twe/paper-trading.html">Paper trading</a> - how simulated orders and fills work',
        '<a href="/twe/token-concepts.html">Token concepts</a> and the <a href="/tokens/index.html">token concept directory</a>',
        '<a href="/twe/tech-stack.html">Technology stack</a> and <a href="/twe/repository-map.html">repository map</a>' ] },
    ] },
  {
    path: 'twe/sandbox-boundary.html', section: 'TWE', title: 'Sandbox boundary',
    lead: 'The line between a design prototype and a real market. TWE stays on the prototype side.',
    sections: [
      { h: 'Deliberately disconnected', p: ['The application does not do any of the following. Keep these capabilities disconnected.'], ul: [
        'Custody assets or connect wallets',
        'Mint NFTs',
        'Deploy or call smart contracts',
        'Publish token concepts',
        'Connect a live market or a test chain',
        'Contact a financial or token provider' ] },
      { h: 'Why it matters', p: [
        'The database tables <code>account_balances</code>, <code>exchange_orders</code> and <code>exchange_trades</code> are demonstration components. They must never be reused as books for customer funds or real assets.',
        'A separate production-readiness checklist can guide planning. It is not authorization to launch.' ] },
    ] },
  {
    path: 'twe/signal-terminal.html', section: 'TWE', title: 'The Signal Terminal look',
    lead: 'A dark, technical visual language for dense market data.',
    sections: [
      { h: 'Palette', ul: [
        'Canvas: deep navy <code>#07131f</code> with raised blue-black panels and thin blue-gray separators',
        'Accent: restrained cyan <code>#20d9ff</code>',
        'Positive sample movement: green <code>#36d7a1</code>',
        'Negative sample movement: coral <code>#ff7184</code>' ] },
      { h: 'Type', p: ['Space Grotesk is used for headings and general interface text. IBM Plex Mono is used for symbols, labels and numeric market data, so columns of numbers line up.'] },
      { h: 'This site', p: ['This documentation site reuses the same palette so the docs feel like part of the product.'] },
    ] },
  {
    path: 'twe/paper-trading.html', section: 'TWE', title: 'Paper trading',
    lead: 'Simulated accounts, simulated orders, simulated fills.',
    sections: [
      { h: 'How it works', p: [
        'New paper accounts are initialized with a simulated USD balance and sample holdings. Market and limit orders update only paper balances, simulated orders and simulated fills.',
        'No order is routed to a live venue. The sample market and order book are illustrative.' ] },
      { h: 'Order types', ul: [
        '<strong>Market</strong> - fills against the illustrative book at the sample price',
        '<strong>Limit</strong> - rests until the simulated price crosses the limit' ] },
      { h: 'Rule of thumb', p: ['If a number on a TWE trading screen ever looks like a quote you could act on, it is not. It is sample data.'] },
    ] },
  {
    path: 'twe/token-concepts.html', section: 'TWE', title: 'Token concept generator',
    lead: 'Deterministic, seeded and private to the signed-in owner.',
    sections: [
      { h: 'Deterministic by design', p: [
        'Token and NFT generators are deterministic from their stored seed. The same seed always yields the same batch, which makes concepts reviewable and reproducible.',
        'Concept batches are private to the signed-in owner. Concepts are not published anywhere.' ] },
      { h: 'The Infinite Horizon / Cosmic Nexus theme', p: [
        'The theme symbolizes boundless liquidity, limitless connectivity and continuous evolution across ecosystems. The <a href="/tokens/index.html">token concept directory</a> on this site shows what a large deterministic batch looks like.' ] },
      { h: 'Identifiers on this site', p: [
        'Every concept carries a <em>concept ID</em> and a <em>draft specification ID</em>. These are labels derived from a hash. They are not contract addresses, and nothing is deployed.' ] },
    ] },
  {
    path: 'twe/nft-planner.html', section: 'TWE', title: 'NFT collection planner',
    lead: 'Plan a collection privately before any minting decision exists.',
    sections: [
      { h: 'Scope', p: ['The planner stores NFT-design batches per owner. It does not mint, connect a wallet or publish anything.'] },
      { h: 'Companion project', p: ['The <a href="/foundry/index.html">Standard Super NFT Foundry</a> is the generation engine used to explore layered collections: weighted traits, exclusions, duplicate prevention, SVG art, narratives and metadata.'] },
    ] },
  {
    path: 'twe/architecture-workspace.html', section: 'TWE', title: 'Architecture workspace',
    lead: 'Owner-scoped planning records with draft-only contract specifications.',
    sections: [
      { h: 'Behavior', ul: [
        'The workspace is owner-scoped',
        'User-entered evidence references are not fetched or verified',
        'The listing review has no live approval or listing state',
        'Contract specifications are draft-only' ] },
    ] },
  {
    path: 'twe/tech-stack.html', section: 'TWE', title: 'Technology stack',
    lead: 'React front end, typed tRPC back end, MySQL persistence.',
    sections: [
      { h: 'Layers', ul: [
        '<strong>Frontend:</strong> React 19, TypeScript, Vite, Tailwind CSS 4, Wouter routing, Lucide icons, TanStack Query and the tRPC client',
        '<strong>Backend:</strong> Node.js and Express, typed tRPC routers, Zod input validation, application services and server-side ownership checks',
        '<strong>Persistence:</strong> MySQL with Drizzle ORM and checked-in migrations. The schema separates users, sample assets, paper balances/orders/trades, token-concept batches, NFT-design batches and architecture-planning records',
        '<strong>Tests:</strong> Vitest and TypeScript checks' ] },
    ] },
  {
    path: 'twe/repository-map.html', section: 'TWE', title: 'Repository map',
    lead: 'Where things live in the source.',
    sections: [
      { h: 'Folders', ul: [
        '<code>client/src/</code> - React routes, UI components, pages and styles',
        '<code>client/public/</code> - public assets, including the watermark',
        '<code>drizzle/</code> - MySQL schema and migrations',
        '<code>server/</code> - tRPC routers and business services',
        '<code>server/_core/</code> - Express runtime and platform adapters',
        '<code>shared/</code> - shared types, enums and deterministic generators' ] },
    ] },
  {
    path: 'twe/run-locally.html', section: 'TWE', title: 'Run the TWE source locally',
    lead: 'Node, pnpm 10.18.0 and a MySQL database.',
    sections: [
      { h: 'Commands', p: ['<pre>corepack enable\npnpm install --frozen-lockfile\npnpm check\npnpm test\npnpm build</pre>'] },
      { h: 'Environment', p: [
        'The original application expects a MySQL database and platform authentication services. Core settings include <code>DATABASE_URL</code>, <code>MANUS_PROJECT_ID</code>, <code>MANUS_JWT_SECRET</code> and <code>MANUS_OAUTH_API_URL</code>.',
        'Never commit <code>.env</code> files or credentials, and never paste them into documents or chats.' ] },
    ] },
  {
    path: 'twe/ai-studio-import.html', section: 'TWE', title: 'Using the source with Google AI Studio',
    lead: 'An import starting point, not a one-click deployment.',
    sections: [
      { h: 'Steps', ul: [
        'Extract the source archive and upload it to a private GitHub repository in your own account. Do not add <code>.env</code> files, database passwords, OAuth secrets or private keys',
        'In AI Studio Build mode choose Add files, then Import from GitHub, and pick the repository',
        'Replace the platform OAuth/session adapter with a supported authentication system (Firebase Authentication is a common choice, but server-side identity verification and user-owned data access must still be implemented)',
        'Choose persistence deliberately: keep a securely hosted MySQL database, or redesign and test the schema for Firestore',
        'Keep secrets in the target platform secret manager. The concept generators are deterministic and need no Gemini API key' ] },
      { h: 'Caveat', p: ['Importing the code does not migrate or provision the existing database, and the platform-bound app does not run unchanged in AI Studio.'] },
    ] },
  {
    path: 'twe/production-readiness.html', section: 'TWE', title: 'Production-readiness planning',
    lead: 'A planning checklist, not authorization to launch.',
    sections: [
      { h: 'Why this page exists', p: ['Turning a sandbox into a real venue is mostly a legal, operational and security project rather than a coding one. The list below is a conversation starter for qualified counsel and compliance staff. It is not legal advice.'] },
      { h: 'Areas to plan', ul: [
        'Regulatory pathway. In the United States, an alternative trading system generally operates under SEC Regulation ATS and through a registered broker-dealer. Other jurisdictions differ. Engage specialist counsel',
        'Asset analysis: whether each listed asset is a security, commodity or something else, per jurisdiction',
        'Identity, sanctions and AML programs using authorized providers',
        'Custody and segregation of customer assets, with independent audit',
        'Market surveillance, fair-access rules and recordkeeping',
        'Settlement lifecycle, reconciliation and failure handling',
        'Security controls, key management, incident response and independent smart-contract audits',
        'Separate production books. The demonstration tables must never become customer ledgers' ] },
    ] },

  /* ---------------------------- FOUNDRY ---------------------------- */
  {
    path: 'foundry/index.html', section: 'Foundry', title: 'Standard Super NFT Foundry',
    lead: 'A runnable Node.js prototype for layered generative collectibles.',
    sections: [
      { h: 'What it does', p: ['The foundry includes a browser dashboard, weighted layer selection, deterministic editions, pre-render trait exclusions, duplicate-fingerprint prevention, SVG rendering, per-edition narratives, ERC-721-style JSON metadata, rarity statistics, a job queue, manifest export and a configurable jurisdiction gate.'] },
      { h: 'Requirements', ul: ['Node.js 22 or newer', 'No third-party runtime dependencies'] },
      { h: 'Explore', ul: [
        '<a href="/foundry/collections.html">Browse the collections</a> generated for this site with the real engine',
        '<a href="/foundry/how-it-works.html">How it works</a>',
        '<a href="/foundry/api.html">API reference</a>',
        '<a href="/foundry/compliance-gate.html">Compliance gate</a>' ] },
    ] },
  {
    path: 'foundry/how-it-works.html', section: 'Foundry', title: 'How the foundry works',
    lead: 'From a theme definition to a finished edition.',
    sections: [
      { h: 'Pipeline', ul: [
        'Load a theme: ordered layers, each with weighted options and an SVG fragment',
        'Seed a random generator from a hash of theme, edition number and attempt',
        'Pick one option per layer by weight',
        'Reject the combination if it hits an exclusion rule or repeats a fingerprint',
        'Render the SVG, write the narrative, write the metadata JSON' ] },
      { h: 'Bounded retries', p: ['Each edition tries up to 2000 seeded attempts. If the trait space is exhausted, or exclusions are too restrictive, the engine throws a clear error. The fix is to add layers or options, or to reduce the batch.'] },
    ] },
  {
    path: 'foundry/determinism.html', section: 'Foundry', title: 'Determinism and fingerprints',
    lead: 'Same inputs, same edition; no two editions alike.',
    sections: [
      { h: 'Determinism', p: ['The seed is the first 32 bits of a SHA-256 hash of <code>theme:edition:attempt</code>, feeding a small Mulberry-style generator. Re-running a job reproduces it exactly.'] },
      { h: 'Fingerprints', p: ['Each edition gets a SHA-256 fingerprint of its sorted <code>trait_type:value</code> pairs. A fingerprint may be used only once per job, which prevents duplicates.'] },
    ] },
  {
    path: 'foundry/rarity.html', section: 'Foundry', title: 'Rarity and weights',
    lead: 'Weights set the odds; rarity statistics report what actually happened.',
    sections: [
      { h: 'Weights', p: ['Every option has a weight. The chance of an option is its weight divided by the total of its layer.'] },
      { h: 'Statistics', p: ['After a job the foundry counts each trait and reports a percentage. On this site, edition pages also show a rarity score (the sum of -ln of each trait frequency) and a tier from Common to Mythic computed over the full generated set.'] },
    ] },
  {
    path: 'foundry/exclusions.html', section: 'Foundry', title: 'Trait exclusions',
    lead: 'Rules applied before a combination is accepted.',
    sections: [
      { h: 'Example', p: ['Matrix Heroes forbids <code>Helmet:Crowned</code> with <code>Armor:Obsidian Exo-Suit</code>, and <code>Power Core:Void Engine</code> with <code>Aura:Green Code</code>. A rule is a list of <code>Layer:Option</code> keys. If every key in a rule is present, the combination is blocked.'] },
      { h: 'Why', p: ['Exclusions keep art coherent, for example avoiding low-contrast pairings.'] },
    ] },
  {
    path: 'foundry/svg-and-narratives.html', section: 'Foundry', title: 'SVG rendering and narratives',
    lead: 'Self-contained 600 x 600 vector art and a short story for every edition.',
    sections: [
      { h: 'SVG', p: ['Layers are drawn in order into a 600 x 600 SVG with the edition number at the bottom. Titles are XML-escaped. SVG fragments are trusted project assets; never accept arbitrary untrusted SVG uploads in production.'] },
      { h: 'Narratives', p: ['The narrative reads the Helmet or Crest, Armor, Power Core or Element, and optional Relic traits, and fills a template. Themes without those layers fall back to generic wording.'] },
    ] },
  {
    path: 'foundry/metadata.html', section: 'Foundry', title: 'Metadata format',
    lead: 'ERC-721-style JSON for every edition.',
    sections: [
      { h: 'Fields', ul: [
        '<code>name</code>, <code>description</code>, <code>image</code>, <code>edition</code>',
        '<code>attributes</code>: a list of <code>{ trait_type, value }</code>',
        '<code>properties</code>: fingerprint, theme and creator (the default creator field is Mauro Lomibao Dar III)' ] },
    ] },
  {
    path: 'foundry/api.html', section: 'Foundry', title: 'API reference',
    lead: 'A small JSON API served by <code>server.js</code>.',
    sections: [
      { h: 'Endpoints', ul: [
        '<code>GET /api/health</code>',
        '<code>GET /api/config</code>',
        '<code>POST /api/jobs</code> with <code>{ theme, count, collectionName, creator, applyThemeExclusions, investmentLinked, jurisdiction, compliance }</code>',
        '<code>GET /api/jobs/:id</code>',
        '<code>GET /api/collections/:id?limit=60</code>',
        '<code>GET /api/export/:id</code> - downloadable JSON manifest' ] },
      { h: 'Running', p: ['<pre>npm start\n# http://localhost:3030\nPORT=3030 MAX_BATCH=500 npm start</pre>', '<code>MAX_BATCH</code> is clamped to 1-5,000; the default is 500.'] },
    ] },
  {
    path: 'foundry/hashlips.html', section: 'Foundry', title: 'HashLips integration direction',
    lead: 'Own small engine today; adapter boundary for tomorrow.',
    sections: [
      { h: 'Position', p: ['HashLips Art Engine 2.0 documents a plugin-based engine with inputs, generators, renderers and exporters, and is described as early development. The older <code>hashlips_art_engine</code> package is a separate legacy codebase.',
        'The foundry implements its own small deterministic layer-swapper so it runs without native image dependencies. It does not embed, and is not affiliated with, HashLips.'] },
    ] },
  {
    path: 'foundry/scaling.html', section: 'Foundry', title: 'Scaling blueprint',
    lead: 'From one process to a distributed pipeline.',
    sections: [
      { h: 'Target shape', p: ['Browser dashboard, API gateway and auth, PostgreSQL job and collection records, a durable Redis/BullMQ queue, stateless generator workers, S3-compatible object storage, a metadata indexer and IPFS pinning.'] },
      { h: 'Today', p: ['The included queue is a single-process demonstration, not a distributed worker system.'] },
      { h: 'About huge page and edition counts', p: ['Page counts and edition counts are bounded by trait space and storage, not by ambition. A theme with 8 layers of 8 options has 16.7 million combinations. At about 3 KB per SVG, one billion editions would need roughly 3 TB. This site is built by a generator with a count setting, so it scales as far as your storage does.'] },
    ] },
  {
    path: 'foundry/compliance-gate.html', section: 'Foundry', title: 'Compliance gate',
    lead: 'A configurable software gate. Not legal advice.',
    sections: [
      { h: 'What it does', p: ['Investment-linked collections can be required to carry attestations: <code>identityVerified</code>, <code>sanctionsScreened</code> and <code>qualifiedInvestorVerified</code>. Configured jurisdictions: US, CA, GB, EU, AU and SG. The restricted list is empty by default.'] },
      { h: 'Notice', p: ['This policy is a software gate only. It does not determine legal eligibility, provide legal advice, or establish that an offering is compliant. Obtain jurisdiction-specific counsel and use an authorized KYC and sanctions provider before production.'] },
    ] },
  {
    path: 'foundry/testing.html', section: 'Foundry', title: 'Testing',
    lead: 'Built-in Node test runner, zero dependencies.',
    sections: [
      { h: 'Run', p: ['<pre>npm test</pre>'] },
      { h: 'Covered', ul: [
        'An edition has a 64-character fingerprint, the right number of traits, and stable hashing',
        'Exclusion rules are respected across editions and the used set grows once per edition',
        'SVG output is valid-looking and the narrative mentions the edition number' ] },
      { h: 'Ideas to add', ul: ['Trait-space exhaustion error', 'XML escaping of titles', 'Weighted distribution within tolerance over many samples'] },
    ] },

  /* ---------------------------- GENERAL ---------------------------- */
  {
    path: 'about.html', section: 'About', title: 'About this site',
    lead: 'Documentation and a generated catalogue for TWE and the NFT Foundry.',
    sections: [
      { h: 'How it was made', p: ['The core pages are written by hand from the project READMEs. The collection, edition, trait and token-concept pages are generated by a build script that runs the real foundry engine, so every edition page has a genuine unique SVG.'] },
      { h: 'Creator', p: ['Project creator field: Mauro Lomibao Dar III.'] },
    ] },
  {
    path: 'roadmap.html', section: 'About', title: 'Roadmap ideas',
    lead: 'Possible next steps.',
    sections: [
      { h: 'Foundry', ul: ['More themes and layers', 'HashLips adapter', 'Durable queue and object storage', 'Metadata pinning (only with explicit approval)'] },
      { h: 'TWE', ul: ['Clean, merged single codebase', 'Security review of contracts and configuration', 'Counsel-led regulatory plan before any live capability'] },
    ] },
  {
    path: 'faq.html', section: 'About', title: 'Frequently asked questions',
    lead: 'Short answers.',
    sections: [
      { h: 'Is TWE a real exchange?', p: ['No. It is a non-production sandbox. ' + SANDBOX] },
      { h: 'Are the token concepts real tokens?', p: ['No. They are deterministic names with draft specification labels. No contract is deployed and no address exists.'] },
      { h: 'Can I mint these NFTs?', p: ['The foundry generates art and metadata only. Minting is not wired in and would need its own review.'] },
      { h: 'Why not 9,999,999,999,999,999,999 pages?', p: ['That is about ten quintillion. It cannot be stored on any real system. The build has a count setting instead.'] },
      { h: 'Where are the secrets?', p: ['Nowhere on this site. Credentials from the uploaded files are intentionally excluded.'] },
    ] },
  {
    path: 'glossary.html', section: 'About', title: 'Glossary',
    lead: 'Terms used across the site.',
    sections: [
      { h: 'Terms', ul: [
        '<strong>ATS</strong> - alternative trading system',
        '<strong>Edition</strong> - one generated NFT in a collection',
        '<strong>Fingerprint</strong> - SHA-256 of an edition sorted trait list',
        '<strong>Exclusion rule</strong> - a combination of traits that may not appear together',
        '<strong>Paper trading</strong> - simulated trading with no real value',
        '<strong>Concept ID</strong> - a label for a generated token concept; not an address',
        '<strong>Draft spec ID</strong> - a label for a draft contract specification; nothing deployed',
        '<strong>Rarity tier</strong> - Common, Rare, Epic, Legendary or Mythic, by percentile of rarity score' ] },
    ] },
  {
    path: 'legal/disclaimer.html', section: 'About', title: 'Disclaimer',
    lead: 'Please read.',
    sections: [
      { h: 'Not an offering', p: ['Nothing on this site is an offer, solicitation, listing, investment advice, legal advice or tax advice. ' + SANDBOX] },
      { h: 'Names', p: ['Generated token names and tickers are creative concepts and may coincide with existing marks or assets. Run a clearance search before any real use.'] },
    ] },
];

export const SANDBOX_TEXT = SANDBOX;
```

## Code 3: `generator/vendor/foundry.js`

The foundry engine (unchanged from the Standard Super NFT Foundry). 37 lines.

```js
import crypto from 'node:crypto';

export function hash(value) { return crypto.createHash('sha256').update(value).digest('hex'); }
const pick = (items, random) => {
  const total = items.reduce((s, x) => s + Number(x.weight || 1), 0);
  let cursor = random() * total;
  for (const item of items) { cursor -= Number(item.weight || 1); if (cursor <= 0) return item; }
  return items.at(-1);
};
function rng(seed) {
  let n = seed >>> 0;
  return () => { n += 0x6D2B79F5; let t = n; t = Math.imul(t ^ t >>> 15, t | 1); t ^= t + Math.imul(t ^ t >>> 7, t | 61); return ((t ^ t >>> 14) >>> 0) / 4294967296; };
}
export function createEdition(themeName, theme, edition, exclusions, used) {
  for (let attempt = 0; attempt < 2000; attempt++) {
    const random = rng(Number.parseInt(hash(`${themeName}:${edition}:${attempt}`).slice(0, 8), 16));
    const traits = theme.layers.map(layer => ({ trait_type: layer.trait_type, option: pick(layer.options, random) })).filter(x => x.option && x.option.name !== 'None');
    const keys = new Set(traits.map(x => `${x.trait_type}:${x.option.name}`));
    const blocked = exclusions.some(rule => rule.every(key => keys.has(key)));
    const fingerprint = hash(traits.map(x => `${x.trait_type}:${x.option.name}`).sort().join('|'));
    if (!blocked && !used.has(fingerprint)) { used.add(fingerprint); return { traits, fingerprint }; }
  }
  throw new Error('Trait space exhausted or exclusions are too restrictive. Add layers/options or reduce the batch.');
}
export function renderSvg(traits, title, edition) {
  const art = traits.map(t => t.option.svg || '').join('\n');
  return `<svg xmlns="http://www.w3.org/2000/svg" width="600" height="600" viewBox="0 0 600 600"><title>${escapeXml(title)}</title>${art}<text x="300" y="565" text-anchor="middle" fill="#ffffff" opacity=".8" font-family="Arial" font-size="17">EDITION ${edition}</text></svg>`;
}
export function narrative(theme, traits, edition) {
  const values = Object.fromEntries(traits.map(t => [t.trait_type, t.option.name]));
  const hero = values.Helmet || values.Crest || 'The Unknown';
  const armor = values.Armor || 'relic armor';
  const power = values['Power Core'] || values.Element || 'an ancient force';
  const relic = values.Relic && values.Relic !== 'None' ? ` Carrying ${values.Relic},` : '';
  return `${hero}, edition ${edition}, emerged from ${theme.description.toLowerCase()} Clad in ${armor.toLowerCase()} and channeling ${power.toLowerCase()},${relic} this champion protects the boundary between imagination and reality.`;
}
function escapeXml(s) { return String(s).replace(/[<>&"']/g, c => ({'<':'&lt;','>':'&gt;','&':'&amp;','"':'&quot;',"'":'&apos;'}[c])); }
```

## Code 4: `generator/package.json`

Marks the generator as an ES module project. 1 lines.

```json
{"type":"module"}
```

## Code 5: `app.config.ts`

Manus project logo metadata. 1 lines.

```ts
export default { logoUrl: "https://files.manuscdn.com/user_upload_by_module/session_file/310519664065017412/IHsZGQnLbkYxmEMH.jpg" }
```

## Code 6: `generator/vendor/layers.json`

Matrix Heroes and Mythic Titans themes (from the foundry config). 220 lines.

```json
{
 "themes": {
  "Matrix Heroes": {
   "description": "Digital guardians born from the code stream.",
   "layers": [
    {
     "trait_type": "Background",
     "required": true,
     "options": [
      {
       "name": "Emerald Grid",
       "weight": 45,
       "svg": "<rect width='600' height='600' fill='#061b18'/><path d='M0 30H600M0 90H600M0 150H600M0 210H600M0 270H600M0 330H600M0 390H600M0 450H600M0 510H600M0 570H600M30 0V600M90 0V600M150 0V600M210 0V600M270 0V600M330 0V600M390 0V600M450 0V600M510 0V600M570 0V600' stroke='#0b735b' opacity='.28'/>"
      },
      {
       "name": "Nebula",
       "weight": 30,
       "svg": "<defs><radialGradient id='bg'><stop stop-color='#572b83'/><stop offset='1' stop-color='#090d25'/></radialGradient></defs><rect width='600' height='600' fill='url(#bg)'/>"
      },
      {
       "name": "Solar Flare",
       "weight": 25,
       "svg": "<defs><linearGradient id='bg' x2='1' y2='1'><stop stop-color='#ffb347'/><stop offset='1' stop-color='#8d2146'/></linearGradient></defs><rect width='600' height='600' fill='url(#bg)'/>"
      }
     ]
    },
    {
     "trait_type": "Aura",
     "required": false,
     "options": [
      {
       "name": "Green Code",
       "weight": 40,
       "svg": "<ellipse cx='300' cy='310' rx='190' ry='230' fill='none' stroke='#39ff9c' stroke-width='12' opacity='.35'/><ellipse cx='300' cy='310' rx='215' ry='255' fill='none' stroke='#39ff9c' stroke-width='3' opacity='.45'/>"
      },
      {
       "name": "Violet Pulse",
       "weight": 30,
       "svg": "<ellipse cx='300' cy='310' rx='195' ry='235' fill='none' stroke='#c77dff' stroke-width='14' opacity='.4'/>"
      },
      {
       "name": "No Aura",
       "weight": 30,
       "svg": ""
      }
     ]
    },
    {
     "trait_type": "Armor",
     "required": true,
     "options": [
      {
       "name": "Quantum Sentinel",
       "weight": 40,
       "svg": "<path d='M205 350L160 510Q300 580 440 510L395 350L350 310H250Z' fill='#526b78' stroke='#b5eaff' stroke-width='8'/><path d='M250 370H350L330 490H270Z' fill='#102a3a' stroke='#59d9ff' stroke-width='6'/>"
      },
      {
       "name": "Solar Paladin",
       "weight": 35,
       "svg": "<path d='M205 350L160 510Q300 580 440 510L395 350L350 310H250Z' fill='#c58b35' stroke='#ffe4a3' stroke-width='8'/><path d='M250 370H350L330 490H270Z' fill='#672e25' stroke='#ffdb77' stroke-width='6'/>"
      },
      {
       "name": "Obsidian Exo-Suit",
       "weight": 25,
       "svg": "<path d='M205 350L160 510Q300 580 440 510L395 350L350 310H250Z' fill='#171b27' stroke='#8c9baa' stroke-width='8'/><path d='M250 370H350L330 490H270Z' fill='#283c50' stroke='#b5d8ff' stroke-width='6'/>"
      }
     ]
    },
    {
     "trait_type": "Helmet",
     "required": true,
     "options": [
      {
       "name": "Visor",
       "weight": 45,
       "svg": "<path d='M205 190Q300 90 395 190L380 315Q300 380 220 315Z' fill='#263d4b' stroke='#bdeaff' stroke-width='9'/><path d='M225 235Q300 195 375 235L365 275Q300 310 235 275Z' fill='#3dffd1' stroke='#c8fff4' stroke-width='5'/>"
      },
      {
       "name": "Crowned",
       "weight": 25,
       "svg": "<path d='M205 190Q300 90 395 190L380 315Q300 380 220 315Z' fill='#b27b2f' stroke='#ffe6a6' stroke-width='9'/><path d='M210 195L190 115L260 160L300 85L340 160L410 115L390 195Z' fill='#e7bd58' stroke='#fff0b7' stroke-width='7'/>"
      },
      {
       "name": "Hooded",
       "weight": 30,
       "svg": "<path d='M175 220Q190 80 300 90Q410 80 425 220L390 370Q300 420 210 370Z' fill='#202337' stroke='#9b8ecb' stroke-width='9'/><path d='M225 240Q300 205 375 240L360 280Q300 310 240 280Z' fill='#a78bfa'/>"
      }
     ]
    },
    {
     "trait_type": "Power Core",
     "required": true,
     "options": [
      {
       "name": "Emerald Reactor",
       "weight": 40,
       "svg": "<circle cx='300' cy='410' r='34' fill='#00ff99' stroke='#d1ffe9' stroke-width='8'/><circle cx='300' cy='410' r='14' fill='#fff'/>"
      },
      {
       "name": "Starheart",
       "weight": 35,
       "svg": "<path d='M300 375L312 400L340 404L320 424L325 452L300 438L275 452L280 424L260 404L288 400Z' fill='#ffdb6e' stroke='#fff2b8' stroke-width='6'/>"
      },
      {
       "name": "Void Engine",
       "weight": 25,
       "svg": "<circle cx='300' cy='410' r='34' fill='#180b35' stroke='#c7a4ff' stroke-width='8'/><circle cx='300' cy='410' r='13' fill='#bd65ff'/>"
      }
     ]
    },
    {
     "trait_type": "Relic",
     "required": false,
     "options": [
      {
       "name": "Chrono Blade",
       "weight": 25,
       "svg": "<path d='M430 365L475 130L495 115L490 390L455 430Z' fill='#c6f4ff' stroke='#fff' stroke-width='5'/><path d='M440 420L490 405L500 430L450 445Z' fill='#d8a84e'/>"
      },
      {
       "name": "Aegis Shield",
       "weight": 25,
       "svg": "<path d='M120 340Q175 310 220 340L210 445Q170 500 130 445Z' fill='#2c8c83' stroke='#b5fff3' stroke-width='8'/>"
      },
      {
       "name": "Nova Staff",
       "weight": 20,
       "svg": "<path d='M470 160L485 500' stroke='#e4c7ff' stroke-width='13'/><circle cx='470' cy='145' r='35' fill='#c27aff' stroke='#f1d7ff' stroke-width='8'/>"
      },
      {
       "name": "None",
       "weight": 30,
       "svg": ""
      }
     ]
    }
   ],
   "exclusions": [
    [
     "Helmet:Crowned",
     "Armor:Obsidian Exo-Suit"
    ],
    [
     "Power Core:Void Engine",
     "Aura:Green Code"
    ]
   ]
  },
  "Mythic Titans": {
   "description": "Ancient champions forged from elemental myth.",
   "layers": [
    {
     "trait_type": "Background",
     "required": true,
     "options": [
      {
       "name": "Temple Dawn",
       "weight": 50,
       "svg": "<rect width='600' height='600' fill='#b85f37'/><circle cx='300' cy='190' r='110' fill='#ffd88a' opacity='.8'/>"
      },
      {
       "name": "Frozen Citadel",
       "weight": 50,
       "svg": "<rect width='600' height='600' fill='#183a57'/><path d='M0 450L150 220L280 450L420 190L600 460V600H0Z' fill='#78b8ce'/>"
      }
     ]
    },
    {
     "trait_type": "Armor",
     "required": true,
     "options": [
      {
       "name": "Dragonforged",
       "weight": 50,
       "svg": "<path d='M205 350L160 510Q300 580 440 510L395 350L350 310H250Z' fill='#9d3d2d' stroke='#ffd18a' stroke-width='10'/>"
      },
      {
       "name": "Frost Titan",
       "weight": 50,
       "svg": "<path d='M205 350L160 510Q300 580 440 510L395 350L350 310H250Z' fill='#8ac9df' stroke='#e4fbff' stroke-width='10'/>"
      }
     ]
    },
    {
     "trait_type": "Crest",
     "required": true,
     "options": [
      {
       "name": "Phoenix Crown",
       "weight": 50,
       "svg": "<path d='M205 210L170 110L260 165L300 70L340 165L430 110L395 210Z' fill='#ffb52e' stroke='#fff0b7' stroke-width='9'/>"
      },
      {
       "name": "Glacier Helm",
       "weight": 50,
       "svg": "<path d='M205 190Q300 90 395 190L380 315Q300 380 220 315Z' fill='#b8efff' stroke='#fff' stroke-width='9'/>"
      }
     ]
    },
    {
     "trait_type": "Element",
     "required": true,
     "options": [
      {
       "name": "Fire",
       "weight": 50,
       "svg": "<path d='M300 370C220 300 320 270 300 200C420 290 400 370 300 455Z' fill='#ff6b32' stroke='#ffe08c' stroke-width='8'/>"
      },
      {
       "name": "Ice",
       "weight": 50,
       "svg": "<path d='M300 370L260 300L300 210L340 300Z' fill='#a7f0ff' stroke='#fff' stroke-width='8'/>"
      }
     ]
    }
   ],
   "exclusions": []
  }
 }
}
```

## Code 7: `foundry.test.js`

Foundry engine tests (your test file; 3 passing). 49 lines.

```js
import test from 'node:test';
import assert from 'node:assert/strict';

import { createEdition, renderSvg, narrative, hash } from '../engine/foundry.js';

const theme = {
  description: 'A realm of brave heroes',
  layers: [
    {
      trait_type: 'Background',
      options: [
        { name: 'A', weight: 1, svg: '<rect width="600" height="600"/>' },
        { name: 'B', weight: 1, svg: '' }
      ]
    },
    {
      trait_type: 'Armor',
      options: [
        { name: 'Gold', weight: 1, svg: '' },
        { name: 'Dark', weight: 1, svg: '' }
      ]
    }
  ]
};

test('creates a unique stable edition', () => {
  const used = new Set();
  const result = createEdition('test', theme, 1, [], used);
  assert.equal(result.fingerprint.length, 64);
  assert.equal(result.traits.length, 2);
  assert.equal(hash('x'), hash('x'));
});

test('exclusion rules are applied before generation', () => {
  const used = new Set();
  for (let i = 1; i <= 3; i++) {
    const result = createEdition('test', theme, i, [['Background:A', 'Armor:Gold']], used);
    const keys = new Set(result.traits.map(t => `${t.trait_type}:${t.option.name}`));
    assert.equal(['Background:A', 'Armor:Gold'].every(key => keys.has(key)), false);
  }
  assert.equal(used.size, 3);
});

test('renders SVG and narrative', () => {
  const edition = createEdition('test', theme, 3, [], new Set());
  const svg = renderSvg(edition.traits, 'Hero', 3);
  assert.match(svg, /<svg/);
  assert.match(narrative(theme, edition.traits, 3), /edition 3/);
});
```

## Code 8: `TODO.md`

Outcome checklist. 14 lines.

```md
# TODO

- [x] Home page with the TWE and NFT Foundry overview, headline statistics and sample edition art
- [x] TWE documentation: sandbox boundary, paper trading, token concepts, tech stack, local setup and production-readiness planning
- [x] NFT Foundry documentation: how it works, fingerprints and determinism, rarity, exclusions, metadata, API, HashLips direction, scaling and compliance gate
- [x] Three NFT collections (Infinite Horizon Cosmic Nexus, Matrix Heroes, Mythic Titans) with overview pages, paginated galleries, rarity tables and trait pages
- [x] 3,612 NFT edition pages, each with unique SVG, rarity rank and tier, trait table, fingerprint, narrative and metadata JSON link
- [x] Token concept directory of 3,000 concept pages, by category, by letter and page by page, each labeled concept-only with no deployed contract
- [x] Client-side search across editions, traits, token concepts and documentation pages
- [x] Site-wide sandbox disclaimer banner, FAQ, glossary, about, roadmap, sitemap and responsive layout
- [x] Static build config declared and checkpoint saved
- [ ] Site published (first deploy hit builder_unavailable; retrying on a new checkpoint)

Rebuild: `cd generator && OUT_DIR=../dist node build.mjs` (COSMIC_EDITIONS, MATRIX_EDITIONS, TOKENS control scale). The committed `dist/` is the published output.
```
