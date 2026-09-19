# Ride Bench

Suspension design tools for the shop, built for the Li'l Or'nge build (1991.5 Dodge D250 Cummins on a 1975 D150 body). Currently: a shared Vehicle Profile, an interactive rear 4-link geometry tool, and a spring/bag rate calculator. More tools (steering geometry, ride height log, 3D interactive view) slot into the sidebar as they're built.

This is a single self-contained `index.html` file — no build step, no server, no dependencies to install. It runs entirely in your browser; nothing is uploaded anywhere, and the only "backend" is your browser's local storage, used to remember your last geometry on your own device.

## Use it right now

Just open `index.html` in any browser — double-click it, or drag it into a browser tab. That's it.

### Getting Started (in-app)

The app opens on a **Getting Started** view by default — a static reference page (no inputs/calculations of its own) that walks through measuring corner weights/CG/tire radius/wheelbase once up front, gives a first-time walkthrough for each tool, and calls out the spots where numbers can still drift apart even with a shared Vehicle Profile (see "How the tools relate" below). It has buttons at the bottom that jump straight into Vehicle Profile or either tool.

## Vehicle Profile (shared data)

**The problem it fixes:** 4-Link Geometry and Spring & Bag Rate both need the same truck-level numbers — wheelbase, tire radius, CG height/position, and corner weights — but originally each tool kept its own separate copy of them. Editing one tool's copy didn't touch the other's, so the two could silently drift apart (e.g. you corner-weigh the truck and update Spring & Bag Rate, but 4-Link Geometry's anti-squat math is still using old numbers).

**The fix:** a dedicated **Vehicle Profile** view holds these numbers exactly once, saved under its own `vehicleProfile-v1` storage key:

- **Wheelbase** and **loaded tire radius** — plain inputs, in inches.
- **CG height above ground** — a plain input too, but flagged as a manual estimate: there's no simple scale trick for it the way there is for corner weight, so it stays whatever you type in (or measure off a tilt table) until you have a better number.
- **Corner weights** (FL/FR/RL/RR) — plain inputs, in lb, ideally from scales.
- **CG, forward of rear axle** — not an input at all. It's computed automatically as `(front total ÷ total weight) × wheelbase`, so it can't drift out of sync with the corner weights that determine it.

Both 4-Link Geometry and Spring & Bag Rate read these values live from Vehicle Profile instead of keeping their own copies, and each shows a small "Using: ..." summary line linking back to Vehicle Profile so you can see at a glance what numbers they're using and jump straight there to change them.

If you were already using an older version of this app, a one-time migration runs the first time you load the new version: it pulls wheelbase/CG height/tire radius out of the old 4-Link tool's storage and corner weights out of the old Spring & Bag Rate tool's storage to populate the new shared profile, so you don't lose numbers you'd already entered. The old per-tool storage keys are left in place afterward (harmless, unused) rather than deleted.

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
- **Wheelbase / CG position / CG height / tire radius** — describe the truck, not the linkage, and now live in the shared **Vehicle Profile** view rather than in this tool. A summary line at the top of the page shows the current values and links straight to Vehicle Profile to edit them.

**Limits:** this is a 2D side-view approximation. It ignores lateral link angles, bushing compliance, and how anti-squat interacts with anti-lift under braking. Use it to compare link layouts and sanity-check geometry before cutting brackets — not as a replacement for measuring the real truck.

### Spring & Bag Rate
Corner weight → wheel rate → ride frequency, for either a coil spring or an air spring (bag). Switch modes with the tabs at the top of the tool.

- **Coil mode** — enter the spring's own rate (lb/in), free length, and the motion ratio at that corner. The tool computes wheel rate (`spring rate × motion ratio²`), ride frequency, and static deflection/sag, and flags deflection that's eating a large share of the spring's free length.
- **Air spring mode** — since a bag isn't a linear spring, its effective rate is estimated from bag pressure, effective piston area, and internal volume (including any plumbed air tank) using the standard adiabatic approximation `k ≈ n·P·A² / V` (n≈1.4, P absolute pressure). A small preset list plugs in ballpark area/volume figures for a few common Slam Specialties/Firestone bags — swap to "Custom" for real spec-sheet or measured numbers.
- **Corner weights** — pulled live from the shared **Vehicle Profile** view for whichever corner is selected as "active." A summary line at the top of the page shows the current corner weights and links straight to Vehicle Profile to edit them.
- **Ride frequency gauge** — a live bar showing where the calculated frequency lands relative to soft/cruiser, sport, and race/off-road ranges, since ride frequency (not spring rate alone) is what actually tracks perceived ride quality.

The page has its own "What do these settings mean?" panel explaining spring rate vs. wheel rate, motion ratio, why ride frequency matters more than spring rate alone, static deflection, and why an air spring's rate changes with pressure while a coil's doesn't.

**Limits:** both modes model a spring/bag in isolation — no bump stops, droop limiters, helper springs, leaf progressive rate, or shock damping, and a real bag's area/volume shift somewhat through travel. Treat the output as a starting point at a given ride height, not a full travel curve.

### Coming next
- **Steering Geometry** — bump-steer and Ackerman checks for the coil/radius-arm front end.
- **Ride Height Log** — track measured ride heights and settings changes over time.
- **3D Interactive View** — a 3D look at the suspension geometry, beyond the current 2D side view.

(The Getting Started view, listed above under "Getting Started (in-app)," is already built — the sidebar roadmap order is Vehicle Profile → Getting Started → 4-Link Geometry → Spring & Bag Rate → Steering Geometry → Ride Height Log → 3D Interactive View.)

## How the tools relate

4-Link Geometry and Spring & Bag Rate both read wheelbase, tire radius, CG height, and corner weights live from the shared **Vehicle Profile** (see above) — so those specific numbers can no longer drift apart between the two tools. That doesn't make everything automatic, though; a few things are still on you to keep consistent by hand:

- **Motion ratio per mode** — Spring & Bag Rate stores motion ratio separately for coil mode and air mode on the same corner. Switching modes doesn't carry the value over — recheck it.
- **Air spring rate is pressure-dependent** — a wheel rate calculated at one bag pressure doesn't hold once you change ride height by changing pressure. Recalculate after adjusting.
- **CG height is a manual estimate** — unlike CG fore-aft (auto-calculated from corner weights), CG height has no simple scale trick and stays whatever you've entered in Vehicle Profile until you measure it some other way.

The in-app **Getting Started** view documents this in more detail, with first-time walkthroughs for each tool.

## App structure

Everything lives in one `index.html` — markup, styles, and logic — so there's no framework or build step to fight. The layout is a sidebar (`.sidebar`) of tools plus a main content area; each tool is a `<section data-view-panel id="view-...">` that the sidebar shows/hides. To add a new tool:

1. Add a `<section class="app" id="view-<name>" data-view-panel hidden>...</section>` inside `<main class="main">`, alongside `view-4link`.
2. Add a `<button class="nav-item" data-view="view-<name>">...</button>` in the sidebar, and drop the `disabled`/`Soon` styling from its placeholder.
3. Give the new tool's script its own IIFE (see the 4-link tool's `<script>` block for the pattern) so its variables don't collide with other tools' scripts.
4. If the tool needs any of wheelbase, tire radius, CG height, or corner weights, read them from `localStorage["vehicleProfile-v1"]` (see the small `loadVehicleProfile()` helper duplicated in the 4-Link and Spring & Bag Rate scripts) rather than adding another copy of those fields — and re-read them on the section's `viewshown` event (dispatched by the nav-switcher whenever that view is shown) so the tool picks up edits made elsewhere in Vehicle Profile.

The 4-link tool's geometry math (four-bar linkage solve, anti-squat calculation) and the spring rate tool's math (wheel rate, ride frequency, adiabatic air spring rate) each live in their own `<script>` block near the bottom of `index.html`, with comments marking each section.

## Known limitations

The 4-link tool is a 2D side-view approximation, and the spring rate tool models a spring/bag in isolation (see above for both). Treat every calculator here as a way to compare designs and sanity-check geometry before cutting metal, not a replacement for measuring the real truck.
