# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is the marketing website for Coffee Suzukotori (コーヒーすずことり), a home-roasted
coffee bean business based in Nagareyama, Chiba, Japan, run by roaster 高柳慎平. The
site is deployed as-is via GitHub Pages (repo `85st/coffee-suzukotori`) — there is no
build step, package manager, framework, or test suite. It is two static, self-contained
HTML files with inline `<style>` and `<script>` tags and no external JS/CSS files.

## Files

- `index.html` — the main landing page (brand story, roasting process, beans CTA,
  market/event section, footer). All photos and the logo are embedded directly as
  `data:image/...;base64,...` URIs inside `<img>` tags, which is why the file is large
  (~1 MB) and individual lines can be hundreds of KB. When reading this file with a
  line-oriented tool, avoid dumping the raw base64 lines (e.g. pipe through `sed` to
  truncate `data:image/...;base64,...` payloads) to avoid wasting context.
- `where.html` — a secondary, lightweight page ("出会える場所" / "Where to find us")
  listing the regular weekly pop-up (OneTable, Matsudo) and upcoming market/event dates.
  It fetches upcoming events from a public Google Calendar via the Calendar API v3 REST
  endpoint, client-side, using a hardcoded `CALENDAR_ID` and `API_KEY` in its inline
  `<script>` (see below).
- `README.md` — one-line project description.

There is no navigation link between `index.html` and `where.html` baked into a shared
header; they are independent pages (linked from Instagram / external channels).

## Working with `index.html` vs `where.html`

- `index.html`'s market/event section renders into `<div class="market-list" id="market-list">`,
  but there is **no script in `index.html` that populates it** — unlike `where.html`,
  which fetches and renders events into `#events` via the Google Calendar API. If asked
  to "wire up" or fix the market section on the top page, this is the gap: either port
  the `where.html` fetch/render logic into `index.html`, or confirm whether market
  listings on the top page are meant to stay static/manual.
- `where.html`'s event script (near the end of the file) is deliberately written to
  degrade gracefully: if `CALENDAR_ID`/`API_KEY` still contain the placeholder `__`
  prefix, or the fetch fails, or the calendar returns no items, it falls back to a
  message pointing to Instagram (`fallback()` function) instead of showing an error.
  Preserve this fallback behavior when editing the calendar integration.

## Google Calendar integration (`where.html`)

- `CALENDAR_ID` and `API_KEY` are plain JS string constants inside the inline `<script>`
  in `where.html` — there is no build-time env injection. The API key is restricted via
  HTTP referrer (`85st.github.io/*`) per the comment above the constants, which is the
  intended mitigation for it being publicly visible in the page source. Don't "fix" this
  by trying to hide the key — that's not how a static GitHub Pages site can work; just
  keep the referrer-restriction assumption in mind if asked about it.
- The calendar must be set to public for the fetch to work (noted in the setup comment
  in `where.html`).
- Events are fetched with `singleEvents=true&orderBy=startTime&maxResults=8` from the
  current time forward, then rendered as `.card` elements matching the same visual
  style as the static "定期の出店" (regular pop-up) card above it.

## Style conventions

- Japanese-first copy; both pages use `lang="ja"`.
- Shared design language: warm, muted "greige/stone" palette defined via CSS custom
  properties in each page's own `:root` block (the two pages do **not** share a palette
  or variable names — `index.html` uses `--bg-base`/`--olive`/etc., `where.html` uses
  `--paper`/`--terra`/etc. — so copy variable names carefully when porting styles
  between the two files rather than assuming a shared design system).
- Fonts are loaded from Google Fonts via `<link>` tags: Shippori Mincho (serif, used for
  headings) and Zen Kaku Gothic New (sans, body text); `index.html` also uses Cormorant
  Garamond for the logo wordmark.
- `index.html` uses an `IntersectionObserver`-based `.reveal` fade/slide-in effect
  triggered on scroll, plus a `.scrolled` class toggle on `#site-header` for a
  background/blur change past 60px of scroll. Keep new sections consistent with this
  pattern (add `class="... reveal"` to sections that should fade in).

## Testing / verifying changes

There is no automated test suite, linter, or build command. Since both files are plain
static HTML, verify changes by opening the file directly in a browser (or serving the
directory with any static file server, e.g. `python3 -m http.server`) and checking the
page visually and in the browser console for JS errors — especially after touching the
`where.html` calendar-fetch script, since failures there are silently swallowed by the
`fallback()` handler and won't show up as visible errors.
