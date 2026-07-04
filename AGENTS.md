# HueForge

Free, fully client-side color palette generator, monetized via Google
AdSense. A deliberate single-purpose spin-out from the AnyConvert multi-tool
site — one keyword-matched domain per tool is the whole SEO strategy here.
**This project must never grow a second tool, a nav to other tools, or a
"more tools" section.** If another tool idea comes up, it gets its own
standalone Astro project under `~/iCloud/website/<name>`, not a page here.

## What it does

Pick a hex color (typed or via the native color picker) and instantly get:

- The main swatch with hex / RGB / HSL values and one-click copy.
- A **complementary** color (180° hue rotation).
- Two **analogous** colors (±30° hue rotation).
- Two **triadic** colors (±120° hue rotation).

Everything is computed with plain JavaScript color math (hex ↔ RGB ↔ HSL
conversion + hue rotation) — no server round-trip, no external color API, no
dependencies.

## Stack

- **AstroJS** (static output, `output: 'static'` in `astro.config.mjs`) +
  **Tailwind v4** (`@tailwindcss/vite` plugin, imported in
  `src/styles/global.css` via `@import 'tailwindcss'` + a `@theme` block for
  the custom color/font tokens).
- **Fonts**: "Fraunces" (display/headings) + "Inter" (body/UI), loaded via a
  Google Fonts `<link>` in `Layout.astro`'s `<head>` with `display=swap`.
- **Zero extra dependencies.** This tool is pure color math — no image/canvas
  libraries, no WASM, nothing beyond Astro + Tailwind + the sitemap
  integration.
- **@astrojs/sitemap** for `sitemap-index.xml` + `robots.txt` pointing at it.
- **Netlify** hosting. Deploy: `netlify deploy --prod --dir=dist` (auth via
  `~/.claude/credentials/netlify.env` → `NETLIFY_AUTH_TOKEN`).

This project intentionally installs ONLY what this one tool needs. It does
NOT include any conversion/media libraries from sibling tools (PixelForge,
ReelShift, StillMotion, ScriptFlow, AnyConvert) — those belong to their own
standalone projects.

## Design system (mandatory, do not deviate)

Premium cream/gold, editorial-serif headings — NOT another SaaS-blue clone,
NOT a generic free-AI-tool site.

- Background: `#FAF6EC` (warm cream)
- Card/surface: `#FFFFFF`, with a warm `#E8DFC8` border (never gray)
- Headings text: `#2B2013` (warm espresso brown, NOT pure black)
- Body text: `#5C4F3D` (warm brown-gray); muted/secondary text `#8A7E68`
- Accent/CTA/links/active states: `#C9982E` (warm gold), hover `#B8860B`
  (darker gold)
- Fonts: **Fraunces** for ALL headings (font-weight 600-700, slightly
  negative letter-spacing for a tightened premium look); **Inter** for body
  text, labels, buttons, and all interactive UI.
- Generous whitespace, soft shadows (never harsh), `rounded-xl`/`rounded-2xl`
  corners on cards/buttons, no gradients, no dark mode toggle (cream-only
  aesthetic is the whole point).

If you touch `src/styles/global.css` or `Layout.astro`, preserve these tokens
exactly. Do not reach for slate/gray/cyan — that was the old AnyConvert look
this tool family was deliberately split away from.

## Structure

- `src/layouts/Layout.astro` — shared shell: brand-only header (no nav to
  other tools by design), footer (About/Privacy/FAQ + copyright), SEO meta
  tags (title/description/canonical/OG/Twitter), Google Fonts link.
- `src/pages/index.astro` — the ONE tool. Hex input + native color picker,
  live main swatch, complementary/analogous/triadic swatch cards, one-click
  copy-to-clipboard, all wrapped in the cream/gold design system. Same color
  math as AnyConvert's `color-picker.astro`. Contains the `escapeHtml()`
  helper — any string interpolated into `innerHTML`-adjacent DOM APIs is
  passed through it defensively, even though the values here are
  machine-generated hex/RGB/HSL strings, not raw user HTML.
- `src/pages/about.astro`, `privacy.astro`, `faq.astro`, `404.astro` —
  required SEO/trust pages, linked from both the header nav and an in-page
  link row on the homepage (not just the footer), per AdSense eligibility
  requirements.
- `public/robots.txt`, `public/ads.txt` (placeholder — replace with the real
  AdSense publisher line once approved), `public/favicon.svg`.

## Security posture

Any user-controlled/user-influenced text rendered via `innerHTML` (or DOM
APIs that behave like it) MUST go through the local `escapeHtml()` helper
first. This is a standing requirement across the whole tool family. In this
tool, the swatch hex/RGB/HSL strings are machine-generated from the typed hex
input, but they still flow through `escapeHtml()` before being set, since
they are derived from user input one step removed. The `<template>` +
`cloneNode` pattern used to render swatch cards avoids raw HTML string
concatenation entirely — the only per-card write is `textContent`, never
`innerHTML`.

## Dev / test

- `npm run dev` — dev server on port **4341** (see `.claude/launch.json`,
  config name `hueforge-dev`). Sibling tools use 4325 (AnyConvert legacy),
  4331 (ReelShift/ScriptFlow), 4332 (StillMotion); 4341 was chosen to avoid
  all of those.
- **iCloud sync hygiene**: `node_modules` is renamed to `node_modules.nosync`
  with a symlink (`node_modules -> node_modules.nosync`) so iCloud doesn't try
  to sync tens of thousands of tiny dependency files. `tsconfig.json`'s
  `exclude` array must list BOTH `"node_modules"` and `"node_modules.nosync"`
  — `tsc`/`astro check` does not treat `.nosync` as an implicit exclusion
  suffix, and without the explicit second entry `astro check` crashes trying
  to type-check into `node_modules.nosync`.
- Test by actually typing a hex value into `#hex-input` (dispatch an `input`
  event) and reading the rendered `#main-hex` / swatch card text back out —
  or by driving `#native-picker` and dispatching `input`. Both paths must
  update all three palette sections.

## Deploy

Netlify site `hueforge-727` (the plain `hueforge` name was already taken by a
sibling build when this project was created — check `.netlify/state.json`
for the actual site ID and the deploy output for the actual URL if this ever
changes).

```
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN=$NETLIFY_API_KEY
netlify deploy --prod --dir=dist
```

## Still needed before this earns anything (manual, Shreyas)

1. Buy/point a real keyword-matched domain.
2. Get real traffic (≥10 daily users per Google Analytics) before applying
   for AdSense.
3. Apply for AdSense, then replace the placeholder line in `public/ads.txt`
   with the real `pub-XXXXXXXXXXXXXXXX` line and add the AdSense script +
   Auto Ads.
4. Submit to Google Search Console + Bing Webmaster Tools.
