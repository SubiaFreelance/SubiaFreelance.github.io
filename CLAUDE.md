# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static marketing site for **Beacon CV Services** (beaconcv.net), served by GitHub Pages from the repo root. The custom domain is configured via `CNAME`. There is **no build step, no package manager, no test suite, and no framework** — every page is hand-authored HTML with inlined CSS and small inline `<script>` blocks.

## Local development

```bash
# Serve the site locally (any static server works; pick one):
python3 -m http.server 8000
# then open http://localhost:8000/
```

Deployment is automatic: pushing to the default branch publishes to GitHub Pages at `https://beaconcv.net/`.

## Site structure

Pages fall into three groups, all at the repo root:

- **Marketing pages** — `index.html` (landing), `about.html`, `contact.html`, `cv-intake.html`, `thank-you.html`.
- **Example/client CV pages** — `tony-subia.html`, `marenco-cv.html`, `rjsubia-cv.html`. Each has a paired downloadable PDF (`Tony_Subia_CV.pdf`, `marenco-cv.pdf`, `Richard_Subia_CV.pdf`) linked from the page's `.dl-bar` button.
- **SEO/infra** — `sitemap.xml`, `robots.txt`, `CNAME`, `favicon.svg`.

When adding a new client CV page, also: (1) add the PDF, (2) add the page to `sitemap.xml`, (3) link it from the `#examples` portfolio grid in `index.html`. The portfolio in `index.html` may reference pages that don't yet exist (e.g. `starnes-cv.html`) — verify before assuming a link is broken.

## Conventions that span multiple files

These patterns recur across pages and are worth preserving when editing:

- **Per-page bespoke styling, no shared stylesheet.** Each HTML file inlines its own `<style>` block and defines a CSS custom-property palette in `:root` (`--ink`, `--paper`, `--accent`, `--warm`, `--muted`, `--rule`, etc.). Client CVs intentionally use distinct color/font combinations — do *not* unify them into a shared stylesheet.
- **Fonts via Google Fonts `<link>`** at the top of `<head>`. Common pairings: `DM Serif Display` + `Source Sans 3` (Beacon brand, Tony's CV), `Playfair Display` + `Lato` or `Source Sans 3` (other CVs).
- **Reveal-on-load animation.** Elements get class `reveal d1`/`d2`/… which fades them in via the shared `@keyframes fadeUp` rule and staggered `animation-delay`s.
- **SEO blocks** on each page: `<title>`, `<meta name="description">`, Open Graph + Twitter card meta, `<link rel="canonical" href="https://beaconcv.net/…">`, and a `<script type="application/ld+json">` JSON-LD block (`ProfessionalService` for the brand, `Person` for CVs). Keep these in sync when copying/editing pages.

## Interactive competency filter (CV pages)

Client CV pages implement a click-to-filter "competencies" feature that's easy to break if you don't know the contract:

1. Competency chips are `<span class="comp-tag" data-comp="KEY" onclick="toggleComp(this)">…</span>`.
2. Highlightable bullets/sentences in the experience section are wrapped in `<span class="hl" data-tags="key1 key2">…</span>` (space-separated keys matching `data-comp`).
3. Job blocks that should dim out when no bullets match get `class="job"` (the script adds/removes `dim` based on whether any descendant `.hl` matched).
4. The shared CSS rules driving the visual state are `body.filtering .hl`, `body.filtering .hl.lit`, and `body.filtering .dim`. Toggling `body.filtering` is the master switch.
5. Print stylesheet (`@media print`) intentionally neutralises the filter: chips render as inline " · "-separated text and `.hl`/`.dim` styling is reset so the printed CV looks clean regardless of filter state.

When adding bullets, set `data-tags` to match the chips you want them to light up under. When adding a new chip, ensure at least one `.hl` element references the new key.

## Forms

- **`contact.html`** posts to **Web3Forms** (`https://api.web3forms.com/submit`) with a hidden `access_key`, hidden `redirect` to `thank-you.html`, and a hidden honeypot checkbox `name="botcheck"` (visually hidden via `.honeypot`). The submit button is disabled on submit and re-enabled after 8s as a fallback.
- **Package pre-selection.** `contact.html` reads `?package=cv|bundle|review` from the URL and pre-selects the matching `<select>` option. Links from `index.html` packages section use `contact.html?package=bundle` / `?package=review`. Keep the map in the inline script in sync if package names/prices change.
- **`cv-intake.html`** is the longer guided intake form (separate from the short contact form).
- **`thank-you.html`** is intentionally `noindex` — it's a post-submit destination, not a discoverable URL. Don't add it to `sitemap.xml`.

## Brand details (frequently referenced in copy)

- Domain: `beaconcv.net` · Contact: `info@beaconcv.net`, `tony@beaconcv.net`
- Packages: CV Writing $350 · CV + Web Presence $500 (featured) · Web Presence Only $200
- Tagline: "Showing the Way Forward"
- Location: West Sacramento, CA

If you change a price, package name, or email, grep for it across `*.html` and `index.html`'s JSON-LD `OfferCatalog` — these strings are duplicated in several places.
