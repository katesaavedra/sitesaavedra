# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page portfolio site for a film director/screenwriter (Ekaterina Saavedra), in Russian. Static HTML/CSS/JS — no build step, no package manager, no test framework. Deployed via GitHub Pages at `https://katesaavedra.github.io/sitesaavedra/` (pushes to `main` go live directly).

## Running locally

There is no dev server or build command. To preview changes, serve the directory with any static file server and open `index.html`, e.g.:

```
npx serve .
```

Then open the printed localhost URL. Just opening `index.html` via `file://` also mostly works but breaks things that require an HTTP origin (fetches, some relative-path edge cases).

## Architecture

Everything lives in two files:

- **`index.html`** — all markup, plus one big inline `(function(){ ... })()` IIFE at the bottom containing all the JavaScript. There is no separate `.js` file and no module system.
- **`styles.css`** — all styling, plain CSS (no preprocessor, no CSS-in-JS).

Content sections are plain `<section id="...">` elements in this order: `hero`, `stats-strip`, `showreel`, `works`, `experience`, `education`, `gallery`, `about`, `contact`.

### Data-driven rendering

Most content is defined as JS data arrays near the top of the IIFE and rendered into empty container `<div>`s at page load. When editing content (adding a project, timeline entry, skill, etc.), edit the data array — do not hand-write repeated markup.

Key arrays/objects (all in `index.html`):
- `WORKS` — portfolio project cards (rendered into `#worksGrid`). Each entry supports: `media` (`'video'`|`'photo'`), `videoSrc`/`modalSrc`/`poster`/`photoSrc`, `gallery` (array of images for a swipeable in-modal gallery, e.g. presentation slides), `watchLinks`/`caseLinks` (external link buttons shown in the modal), `badges`/`badgeIcon`, `tagIcon`, `noPlayBtn`, `mediaCaption`, `note`, `contactLinkLabel`, `roles` (used by the Режиссура/Сценарии filter toggle).
- `FILTERS` — genre filter chips above the works grid.
- `ROLE_FILTERS` / `roles` on each work — the "Режиссура / Сценарии" toggle above the genre filters; filtering combines both (see `renderWorks`).
- `TIMELINE` — experience timeline entries (year + bullet items), rendered inside `#experience` over a background photo with a hand-drawn SVG "wave" line (see below).
- `EDUCATION` — school/course cards; entries with a `diploma` field open a PDF in the shared modal (`openDiploma`) instead of a new tab.
- `SKILLS_PRIMARY` — flip-style skill cards (`label` + `proof` shown on hover) rendered into `.skills-grid`.
- `SKILLS_SECONDARY`, `SKILLS_SOFT` — smaller skill/quality lists styled with the hand-drawn "pencil" underline (`.underline-sketch` / `.skill-pill.secondary`).
- `GALLERY_RAW` — the horizontal-scroll "бэкстейдж" photo/video strip.
- `NAV_DEFS` — top nav links, driven by `scrollToId(id)` smooth-scroll.

### Shared work/diploma modal

There's a single modal (`#workModal` / `#modalBox` / `#modalMedia` / `#modalInfo`) reused for both project details (`openWork(work)`) and diploma PDFs (`openDiploma(title, pdfSrc)`). `openWork` branches on the work's fields to decide what to render in `modalMedia` — image gallery, video, single photo, or placeholder — then builds `modalInfo`'s HTML from the work's title/year/tags/role list/links. `openDiploma` toggles a `.diploma-mode` class on `modalBox` for a portrait A4-ish layout with the info panel hidden.

### Filenames and `encPath`

All media lives in `uploads/` (git-ignored: a few oversized raw video sources, and `*.mp4.part`; see `.gitignore` for the exact exclusions). Many filenames contain Cyrillic characters and spaces. Never hardcode a raw path in an `href`/`src` in JS — always run it through `encPath(path)` (defined near the top of the IIFE), which percent-encodes each path segment. Template strings that build markup for images/videos/PDFs already do this; follow the same pattern for new ones.

### Scroll-driven effects

- `#experience` has a decorative SVG line (`#timelineWavePath`) that reacts to scroll position and mouse movement to produce a moving "curl" in an otherwise straight line — see `updateTimelineWave()`. This is deliberately not a real 3D/physics effect, just a parametrized sine-based path recomputed every animation frame.
- The sticky nav (`#siteNav`) hides on scroll-down and reappears on scroll-up via `handleScroll()`.

### Recurring visual patterns worth reusing

- **"Sketchy" rotated highlight behind heading words**: `.hl-word::before` (see `#education h2 .hl-word::before` etc.) — a solid accent-colored rectangle, slightly rotated, sitting behind part of a heading word. Section-specific selectors override the rotation/position per heading.
- **Hand-drawn pencil underline**: `.underline-sketch` (and reused by `.skill-pill.secondary`) — an inline SVG data-URI with `feTurbulence`/`feDisplacementMap` for a wobbly underline instead of a plain `border-bottom`.
- **Scattered/overlapping photo stacks**: `.stats-storyboard` and `.extra-storyboard` — absolutely positioned `<img>`s with per-child rotation/offset to look like loosely stacked printed photos.

## Working with the user

The user (site owner, non-developer) drives changes conversationally and reviews visually — she does not read code. When making CSS/JS/layout changes of any real visual weight, verify by actually rendering the page (headless browser screenshot) rather than reasoning about CSS in the abstract; several past bugs (stacking-context z-index issues, `align-items: stretch` distorting sibling heights, a `width`/`viewBox` mismatch on an SVG) were only caught this way.

Playwright is not pre-installed in this repo; past sessions installed it ad hoc into the scratchpad directory (`npm install playwright@1 --no-save`, then `npx playwright install chromium` if the browser binary is missing) to take verification screenshots. It is not a project dependency — don't add it to a manifest that doesn't exist.
