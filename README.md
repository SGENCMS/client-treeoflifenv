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
| Stage 5 — bundle audit, re-run on **this published tree** | **12/13** |
| Stage 5 — Gate 13, runtime off-origin requests | **PASS — 0 escapes** |
| Stage 5 — Layer 2 assertions | **1 hard**, 2 soft → overall FAIL |

The single remaining gate failure is Gate 1 (folder structure), and it fails *because of how
this repo is published*, not because the capture is wrong: Gate 1 wants the pipeline's
`project/` subdirectory, which is flattened to the repo root so GitHub Pages can serve
`/client-treeoflifenv/` directly.

Gate counts in this file have been wrong before — an earlier revision claimed 12/12, a figure
carried over from the pre-flatten bundle. They are now re-derived by running
`clone-stage-5-audit.mjs` against the published tree rather than copied from anywhere.

### Gate 13 is new, and it exists because of this bundle

The pipeline gained a **runtime** off-origin gate as a direct result of auditing this preview.
Gates 1–12 are static, and static analysis is structurally blind to the defect that shipped
here: the production URLs are not literals in the markup: they are assembled at runtime from
inline config (`Defaults.base_url`, `sgcom.config.cart.urls`) and consumed by a script in
another file. This preview passed all twelve static gates while fetching the client's
production origin on load.

Gate 13 renders the bundle headless, exercises it, and fails on any programmatic off-origin
request. Off-origin *navigation* is aborted rather than counted, because a cloned page's nav
links legitimately point at the real site. Only the Google Fonts hosts are allowlisted, and
it fails closed if the browser cannot run.

Gate 10 was also repaired: it had been passing a literal `true` and asserting nothing, which
is how a live tag-manager `<noscript>` iframe shipped past it.

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

This is a public copy of a client's page, so six things were changed from what was
captured. They are deliberate, and they are the only edits to the source markup.

1. **`noindex, nofollow`.** The source served `index, follow, max-image-preview:large`.
   A public duplicate of a client's page must not compete with the client's own site in
   search results.
2. **Tracking removed.** The source carried a Google Tag Manager `<noscript>` iframe
   pointing at a live tag-manager container; left in place it would have fired the
   client's real analytics on every preview visit. The tag is gone, and the orphaned
   GTM / Microsoft Clarity payloads (~0.9 MB) were pruned from `_xorigin/`.
3. **Subresource Integrity stripped from vendored scripts.** jQuery and Bootstrap were
   rewritten to local `_xorigin/` paths but kept their original `integrity=` and
   `crossorigin="anonymous"` attributes. Against a local file those force a CORS request
   that fails, so **neither library executed** — the page looked right but was inert.
   Removing the attributes restores them; verified at runtime (`window.jQuery` 3.7.0,
   `window.bootstrap` 12 components).
4. **All programmatic calls to the client's production API are blocked**, by a global
   transport guard in the first `<script>` of `index.html` that wraps `fetch`,
   `XMLHttpRequest` and `navigator.sendBeacon` and rejects anything off-origin.
   The first attempt at this fix guarded one function (`ajax()` in `dispenza.js`) and was
   insufficient — at least five other call sites were still open, including
   `ecommerce.plugin.js:71`, which fetched `/do_shopping/cart_state` from the client's
   production server *while the browser stayed on the preview*. The production URLs are
   injected as inline config in `index.html`, not by the guarded module, so a per-file
   guard could never have covered them. See "A correction" below.
7. **`no-referrer` referrer policy.** Outbound navigation (nav links, the search form)
   still goes to the client's real site by design — but the preview's URL was travelling
   as the `Referer` and landing in the client's own GA4 and Clarity as an unexplained
   traffic source. This stops that without changing what the links do.
5. **Social/unfurl metadata rewritten.** The source `og:`/`twitter:` tags carried the
   client's real title, `og:site_name`, description and hotlinked logo, with `og:url`
   pointing at `treeoflifenv.com`. `noindex` governs search indexers only — it does **not**
   stop link-preview crawlers, so pasting this URL into Slack, Teams, Facebook, X, LinkedIn
   or iMessage rendered a card indistinguishable from a share of the official dispensary.
   The tags now identify the page as an unofficial preview and point at this preview's own
   URL. The two `schema.org` JSON-LD blocks (`Organization`, `WebSite`), which asserted the
   client's business identity, were removed for the same reason. None of this is rendered,
   so the pixel match is unaffected.
6. **`audit.json` paths made repo-relative.** It is served publicly and was disclosing the
   build machine's absolute Windows paths.

`<link rel="canonical">` still points at `https://treeoflifenv.com/` — correct, since the
client's page is the canonical original.

## A correction

This README has now been wrong twice about the same thing, and both errors are recorded
here rather than quietly edited.

**First error.** It stated "the clone does not phone home." The cart request fails visibly
in the browser console, which made it look purely local. It was not: the request reached
the client's production origin, was answered `200` at the preflight, and minted a session
cookie, once per page view. *A CORS error in the console means the response was refused,
not that the request was never sent.*

**Second error.** After guarding `ajax()` in `dispenza.js`, it claimed the preview made
"zero off-origin requests". That was verified on an *untouched page load only*. Driving the
page broke it immediately — `window.sgcom.cart.refresh()` still reached
`https://treeoflifenv.com/do_shopping/cart_state` via a completely different file, while the
browser stayed on the preview. The guard also had a real bug: it resolved against
`location.href` while `fetch()` resolves against `document.baseURI`. *Verifying a network
claim against a page you never interact with proves almost nothing.*

The current guard is global, sits at the transport layer, and is verified by exercising 73
controls on the page and asserting zero programmatic off-origin requests — not by loading
the page and looking at the console.

**Scope, stated precisely:** *programmatic* off-origin requests are blocked. User-initiated
*navigation* is not — clicking a nav link or submitting the search box still takes you to
the client's real site, which is the intended behaviour for a preview of a page whose whole
job is to route visitors there. Those navigations do reach the client's server. The
`no-referrer` policy (change 7) keeps the preview's URL out of their analytics.

## Known limits

**The cart drawer stays empty.** It hydrates from `/dispenza/ajax/cart_html` on a backend
that does not exist here. The request is now refused locally by the same-origin guard
(change 4), so the drawer renders as an inert empty shell and nothing leaves the browser.
No static bundle can satisfy it.

## Residual risks not addressed here

These need a decision rather than a patch, and are recorded so they are not mistaken for
oversights:

- **No on-page disclosure.** Nothing *rendered* tells a visitor this is not the official
  site — the disclosure lives in `<head>` metadata and in this README. A visible banner
  would fix it but would break the pixel-fidelity that is the point of the artifact. The
  alternative is hosting previews privately instead of on a public URL.
- **The public org exposes the client roster.** `SGENCMS` currently hosts seven public
  `client-*` Pages sites naming other dispensaries. That is an org-level hosting decision,
  not something this repo can fix.
- **No framing or CSP headers.** GitHub Pages cannot set response headers, so this
  pixel-accurate replica can be embedded in an iframe by anyone. The source site sets
  `frame-ancestors 'self'`; this preview cannot.
- **The commit metadata is public**, including the committer email and a build-session
  trailer.
- **The pre-remediation commit is still public.** Remediation here was a forward commit, so
  the original `0915074` remains fetchable — including the `audit.json` with absolute build
  paths and the impersonating `og:` tags. For anything treated as a *disclosure*, rewriting
  `HEAD` is not removal; only a history rewrite and force-push (or deleting and recreating
  the repo) removes it. That is a destructive operation on a published repo and is left as
  a deliberate decision.
- **`<link rel="canonical">` still points at the client's page.** This is the standard
  duplicate-content mitigation and it benefits the client, but it is also an identity
  assertion pointing away from this preview. Kept deliberately; flagged because it cuts
  both ways.

## Vendored file integrity

SRI was removed (change 3), so nothing pins these files any more. Recorded here so any
later divergence is a one-command `sha256sum -c` away:

```
e4fd49181388c48ec5040bd3fe66f57c29c8e67fcd8502b3354b96ec7ab47cc7  _xorigin/cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js
b8da2c25347b69ad3d7b5b8346725c2b0d06e86712de316e9b0578528e9d4f40  _xorigin/cdnjs.cloudflare.com/ajax/libs/font-awesome/7.3.0/webfonts/fa-brands-400.woff2
f4f5cc8867d30647f0ad918c01599fa8aa9657c39eb55f0fc610120986544385  _xorigin/cdnjs.cloudflare.com/ajax/libs/font-awesome/7.3.0/webfonts/fa-regular-400.woff2
47b1a018f969189c59b87e9d23f9304db54dc9bdcecf00b216c23515f86826e4  _xorigin/cdnjs.cloudflare.com/ajax/libs/font-awesome/7.3.0/webfonts/fa-solid-900.woff2
17256d81225ece54f4e0e3b3711997d094257197f6ac3e2e6ece9cfafa9f9e39  _xorigin/cdnjs.cloudflare.com/ajax/libs/font-awesome/7.3.0/webfonts/fa-v4compatibility.woff2
d8f9afbf492e4c139e9d2bcb9ba6ef7c14921eb509fb703bc7a3f911b774eff8  _xorigin/cdnjs.cloudflare.com/ajax/libs/jquery/3.7.0/jquery.min.js
```

Verified byte-identical to a fresh fetch from the origin CDNs at publish time.

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

## License / ownership

Clone of client source content, for preview and development use. No license is granted;
all rights in the captured content remain with the original site owner.
