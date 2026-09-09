# client-treeoflifenv

Static preview bundle of **treeoflifenv.com** (Tree of Life — Nevada cannabis dispensary)
— cloned landing page, captured by the clone-site pipeline (Clone Stages 1–5).

**Preview: https://sgencms.github.io/client-treeoflifenv**

Pure static. No build step, no dependencies, no backend. Open `index.html` or use the
preview link above.

## Pages

| File | Source |
| --- | --- |
| `index.html` | `https://treeoflifenv.com/` |

### What this page actually is

`treeoflifenv.com` is **not a full homepage**. It is a 21+ age-gate / location-select
splash — about 578 characters of body text — whose job is to send visitors to one of two
separate dispensary sites:

- `https://lasvegas.treeoflifenv.com`
- `https://northlasvegas.treeoflifenv.com`

Those two subdomains were **not** cloned. Every outbound link here (ORDER NOW, HOME,
REWARDS, CHARITABLE EFFORTS, cart, checkout) points at live `treeoflifenv.com` or those
subdomains, so clicking one leaves this preview and lands on the real site.

## Verification

Captured and checked by the pipeline, not by eye.

| Gate | Result |
| --- | --- |
| Stage 4 — pixel diff vs live source, 6 viewports | **PASS 6/6** |
| Stage 5 — bundle audit, 12 acceptance gates | **12/12 PASS** |
| Stage 5 — Layer 2 assertions | **1 hard**, 2 soft → overall FAIL |

Per-viewport pixel match against the live site (gate is 0.95):

| Viewport | Match |
| --- | --- |
| 1440×900 large desktop | 100.000% |
| 1280×800 standard desktop | 100.000% |
| 1024×768 large tablet | 100.000% |
| 768×1024 small tablet | 100.000% |
| 430×932 large phone | 100.000% |
| 390×844 standard phone | 100.000% |

A flat 100.000% is unusual and is explained by the subject: a short, static splash page
with no fluid imagery or JS-driven layout. The same 6/6 was reproduced with the bundle
served from a `/client-treeoflifenv/` subpath, matching how GitHub Pages actually serves
it — **0 subpath 404s**. Full detail in `audit.json`.

### The one failing assertion

Stage 5 reports **12/12 acceptance gates green** but fails overall on a single Layer 2
hard issue: `assets/defaults/js/jquery.min.js` is a framework import, and the pipeline
spec (§L.1) says frameworks are stripped and their behaviors reauthored as vanilla IIFE
JS (§L.2).

This is a **spec conflict, not a defect**. The source site is built on jQuery 3.7.0 +
Bootstrap 5.3.8, and this page's behaviors (age gate, location select, cart drawer,
search) are not covered by the pipeline's five IIFE templates. Passing §L.1 would mean
hand-reauthoring those four behaviors. The bundle ships as a faithful clone instead, and
the audit records the conflict honestly rather than suppressing it.

## Changes made to the capture

This is a public copy of a client's page, so three things were changed from what was
captured. They are deliberate, and they are the only edits to the source markup.

1. **`noindex, nofollow`.** The source served `index, follow, max-image-preview:large`.
   A public duplicate of a client's page must not compete with the client's own site in
   search results.
2. **Tracking removed.** The source carried a Google Tag Manager `<noscript>` iframe
   pointing at a live tag-manager container; left in place it would have fired the
   client's real analytics on every preview visit. The tag is gone, and the orphaned
   GTM / Microsoft Clarity payloads (~0.9 MB) were pruned from `_xorigin/`. The clone
   does not phone home.
3. **Subresource Integrity stripped from vendored scripts.** jQuery and Bootstrap were
   rewritten to local `_xorigin/` paths but kept their original `integrity=` and
   `crossorigin="anonymous"` attributes. Against a local file those force a CORS request
   that fails, so **neither library executed** — the page looked right but was inert.
   Removing the attributes restores them; verified at runtime (`window.jQuery` 3.7.0,
   `window.bootstrap` 12 components).

`<link rel="canonical">` and `og:url` already pointed at `https://treeoflifenv.com/` in
the source and were left unchanged.

## Known limits

**The cart drawer cannot load.** It fetches `/dispenza/ajax/cart_html` from a live backend
that does not exist here, and cross-origin the request is refused. This is the only
runtime error the page produces. No static bundle can satisfy it.

**The search box leaves the preview.** The header search is a native GET form with
`action="https://treeoflifenv.com/search"`, so submitting it navigates to the live site's
search results — the same behaviour as every other outbound link on the page. It was left
intact rather than stubbed, because nothing is submitted silently and no personal data is
involved.

## Tech notes

- All CSS is consolidated into a single sibling `chrome.css` (706 KB, from 15 source
  stylesheets) rather than an inline `<style>` block — the page's stylesheet is large
  enough that inlining it would bloat the HTML for no gain in self-containment.
- The page uses **system fonts**, not a Google Fonts family; only the font preconnect
  hints from the source `<head>` remain. Font Awesome 7.3.0 webfonts, jQuery 3.7.0 and
  Bootstrap 5.3.8 are vendored under `_xorigin/`, so the page renders correctly offline.
- **`.nojekyll` is required.** Without it GitHub Pages drops every `_`-prefixed directory,
  which would silently 404 jQuery, Bootstrap and every Font Awesome webfont.
- Asset paths mirror the source URL structure verbatim.

## Ownership

Clone of client source content, for preview and development use.
