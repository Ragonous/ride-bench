# Ride Bench

Suspension design tools for the shop. Currently: an interactive rear 4-link geometry tool for the Li'l Or'nge build (1991.5 Dodge D250 Cummins on a 1975 D150 body). More tools (spring/bag rate, steering geometry, ride height log) slot into the sidebar as they're built.

This is a single self-contained `index.html` file — no build step, no server, no dependencies to install. It runs entirely in your browser; nothing is uploaded anywhere, and the only "backend" is your browser's local storage, used to remember your last geometry on your own device.

## Use it right now

Just open `index.html` in any browser — double-click it, or drag it into a browser tab. That's it.

## Host it for free so you can get to it from any device

**GitHub Pages** — free static hosting straight from this repo, no server or account beyond GitHub itself:

1. Create a new repository on GitHub (e.g. `ride-bench`) and push this folder to it:
   ```bash
   git remote add origin https://github.com/<your-username>/ride-bench.git
   git branch -M main
   git push -u origin main
   ```
2. On GitHub, go to the repo's **Settings → Pages**.
3. Under "Build and deployment," set **Source** to "Deploy from a branch," pick the `main` branch and `/ (root)` folder, then save.
4. GitHub gives you a URL like `https://<your-username>.github.io/ride-bench/` within a minute or two. Bookmark it — that's your app, live on the web, for free, forever, with no dependency on any chat session.

Any time you want to change something, edit `index.html`, commit, and push — the live page updates automatically.

## Current tools

### 4-Link Geometry
Drag the link pivots, run the travel slider, watch anti-squat and instant center update live. Solves the actual four-bar linkage — both links keep their real fixed length through simulated travel, not just a redrawn picture.

The page has a "What do these settings mean?" panel built in (open by default) explaining every control:

- **Fu / Au** — the upper link's frame-side and axle-side mounting points.
- **Fl / Al** — the lower link's frame-side and axle-side mounting points.
- **Instant center (IC)** — where the extended lines of the two links cross. A geometric construction, not a physical part, that shows how the axle "wants" to move at that instant.
- **Anti-squat %** — how much of the rear's squat under acceleration the geometry cancels out. 100% cancels it fully; over that starts to jack the rear up instead.
- **Travel slider** — re-solves the actual four-bar linkage as you compress or extend the suspension, so you see how anti-squat and IC shift through the range, not just at rest.
- **Pinion angle change** — how much the axle housing rotates through travel, relative to static ride height.
- **Wheelbase / CG position / CG height / tire radius** — describe the truck, not the linkage. These feed the anti-squat calculation; the defaults are placeholders until you corner-weigh the truck.

**Limits:** this is a 2D side-view approximation. It ignores lateral link angles, bushing compliance, and how anti-squat interacts with anti-lift under braking. Use it to compare link layouts and sanity-check geometry before cutting brackets — not as a replacement for measuring the real truck.

### Coming next
- **Spring & Bag Rate** — corner weight → wheel rate → ride frequency, for coil, leaf, and air spring setups.
- **Steering Geometry** — bump-steer and Ackerman checks for the coil/radius-arm front end.
- **Ride Height Log** — track measured ride heights and settings changes over time.

## App structure

Everything lives in one `index.html` — markup, styles, and logic — so there's no framework or build step to fight. The layout is a sidebar (`.sidebar`) of tools plus a main content area; each tool is a `<section data-view-panel id="view-...">` that the sidebar shows/hides. To add a new tool:

1. Add a `<section class="app" id="view-<name>" data-view-panel hidden>...</section>` inside `<main class="main">`, alongside `view-4link`.
2. Add a `<button class="nav-item" data-view="view-<name>">...</button>` in the sidebar, and drop the `disabled`/`Soon` styling from its placeholder.
3. Give the new tool's script its own IIFE (see the 4-link tool's `<script>` block for the pattern) so its variables don't collide with other tools' scripts.

The 4-link tool's geometry math (four-bar linkage solve, anti-squat calculation) is in the `<script>` block near the bottom of `index.html`, with comments marking each section.

## Known limitations

The 4-link tool is a 2D side-view approximation — see above. Treat every calculator here as a way to compare designs and sanity-check geometry before cutting metal, not a replacement for measuring the real truck.
