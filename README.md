# Ride Bench

Suspension design tools for the shop, built for the Li'l Or'nge build (1991.5 Dodge D250 Cummins on a 1975 D150 body). Currently: a shared Vehicle Profile, an interactive rear 4-link geometry tool, a spring/bag rate calculator, a front-end steering (bump steer) geometry tool, and a ride height log. More tools (a 3D interactive view) slot into the sidebar as they're built.

This is a single self-contained `index.html` file — no build step, no server, no dependencies to install. It runs entirely in your browser; nothing is uploaded anywhere, and the only "backend" is your browser's local storage, used to remember your last geometry on your own device.

## Use it right now

Just open `index.html` in any browser — double-click it, or drag it into a browser tab. That's it.

### Getting Started (in-app)

The app opens on a **Getting Started** view by default — a static reference page (no inputs/calculations of its own) that walks through the order to use the app in: fill in Vehicle Profile first, then 4-Link Geometry, then Spring & Bag Rate, plus a reminder to keep Vehicle Profile current as the build progresses (see "How the tools relate" below). It has buttons at the bottom that jump straight into Vehicle Profile or either tool.

## Vehicle Profile (shared data)

**The problem it fixes:** 4-Link Geometry and Spring & Bag Rate both need the same truck-level numbers — wheelbase, tire radius, CG height/position, and corner weights — but originally each tool kept its own separate copy of them. Editing one tool's copy didn't touch the other's, so the two could silently drift apart (e.g. you corner-weigh the truck and update Spring & Bag Rate, but 4-Link Geometry's anti-squat math is still using old numbers).

**The fix:** a dedicated **Vehicle Profile** view holds these numbers exactly once, saved under its own `vehicleProfile-v1` storage key:

- **Wheelbase**, **loaded tire radius**, and **track width** — plain inputs, in inches. Track width (side-to-side distance between wheel centerlines) was added specifically for the Steering Geometry tool, but lives here rather than in that tool since it's a vehicle-level fact any future front-end tool would also want.
- **CG height above ground** — a plain input too, but flagged as a manual estimate: there's no simple scale trick for it the way there is for corner weight, so it stays whatever you type in (or measure off a tilt table) until you have a better number.
- **Corner weights** (FL/FR/RL/RR) — plain inputs, in lb, ideally from scales.
- **CG, forward of rear axle** — not an input at all. It's computed automatically as `(front total ÷ total weight) × wheelbase`, so it can't drift out of sync with the corner weights that determine it.

4-Link Geometry, Spring & Bag Rate, and Steering Geometry all read these values live from Vehicle Profile instead of keeping their own copies, and each shows a small "Using: ..." summary line linking back to Vehicle Profile so you can see at a glance what numbers they're using and jump straight there to change them.

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

### Steering Geometry
A front-view bump steer check for a solid-axle, track-bar/drag-link front end (steering box → pitman arm → drag link → steering arm, with the axle located side-to-side by a track bar/panhard rod). Drag the pivots, run the travel slider, watch the bump steer number move — reuses the 4-Link tool's four-bar linkage solve (circle-circle intersection) applied to a front-view problem instead of a side-view one.

- **Tf / Ta** — the track bar's frame-side and axle-side mounting points. As the suspension cycles, the axle arcs around Tf, since Ta is stuck the track bar's fixed length away.
- **Df / Da** — the drag link's frame-side end (effectively the pitman arm output) and its axle-side end (the steering arm on the knuckle).
- **Bump steer error** — the horizontal (toe) displacement of Da versus where it would land if it had swung through an identical arc to Ta's (same rotation, same center at Tf). Near zero means the track bar and drag link are tracking each other well through travel; a large or growing value means the front end steers itself as it cycles — no driver input required.
- **Bump steer risk, full travel range** — the worst bump steer error seen anywhere across the whole travel slider sweep, not just at the current position, flagged good/warn/bad the same way the 4-Link tool flags anti-squat.
- **Travel slider** — re-solves both arcs together: the track bar rotates around Tf by the travel angle, fixing a new axle-side point Ta; the drag link and the rigid axle (a fixed distance from Ta to Da) are then solved to find where Da lands, exactly the same circle-intersection technique the 4-Link tool uses for its upper/lower links.

The page has its own "What do these settings mean?" panel explaining what a track bar and drag link do, why their geometry needs to match, what bump steer feels like on the road (wandering, small constant corrections, a tug on the wheel when a front tire drops into a dip mid-corner), and that the fix is usually relocating a frame-side pivot so the two links' lengths and angles converge — something to iterate on here before cutting brackets, not weld-and-hope.

**Limits:** this is a 2D front-view approximation, same spirit as the 4-Link tool's side-view one. It ignores bushing compliance, steering box lash, and the rest of the steering linkage's Ackermann/tie-rod geometry, and treats "identical arc to the track bar" as the standard for zero bump steer.

### Ride Height Log
A running record of measured ride height at all four corners, so changes (new springs, a bag pressure adjustment, added weight, worn bushings) can be checked against what was actually measured before — instead of relying on memory of "it used to sit higher."

- **Log a measurement** — a date (defaults to today), a height reading for each corner (in, measured however you define ride height — fender lip to ground, for example), and a free-text note (e.g. "after installing new rear leaf pack" or "bags at 70psi"). Add entry appends it to the log and saves immediately.
- **Logged entries** — a table of every entry, most recent first, with a delete button per row. Deleting takes two clicks (the button turns into "Confirm delete?" and reverts on its own after a few seconds) so a misclick can't quietly erase history.
- **Most recent reading** — average front height, average rear height, and the front-minus-rear difference (rake) for the latest entry, so a nose-high or nose-low trend is visible at a glance without reading every row.
- **Height over time chart** — once there are 2+ entries, a small hand-drawn SVG line chart (one line per corner, in the app's existing color tokens) plots each corner's height across all logged dates.

The page has its own "What do these settings mean?" panel explaining why a consistent measuring reference point matters, what a front/rear height difference (rake) affects (headlight aim, aero, sometimes handling balance), and why it's worth logging a baseline now — even with rough numbers — and logging again after every suspension-related change.

**Limits:** it's a log, not a calculator — there's no cross-check against the other tools or unit conversion, just the simple front/rear split above. It's only as useful as the discipline of measuring the same way and logging consistently.

### Coming next
- **3D Interactive View** — a 3D look at the suspension geometry, beyond the current 2D side view.

The sidebar's full roadmap order is Vehicle Profile → Getting Started → 4-Link Geometry → Spring & Bag Rate → Steering Geometry → Ride Height Log → 3D Interactive View.

## How the tools relate

4-Link Geometry, Spring & Bag Rate, and Steering Geometry all read truck-level numbers live from the shared **Vehicle Profile** (see above) — so wheelbase, tire radius, track width, CG height, and corner weights can no longer drift apart between those three tools. That doesn't make everything automatic, though; a few things are still on you to keep consistent by hand:

- **Motion ratio per mode** — Spring & Bag Rate stores motion ratio separately for coil mode and air mode on the same corner. Switching modes doesn't carry the value over — recheck it.
- **Air spring rate is pressure-dependent** — a wheel rate calculated at one bag pressure doesn't hold once you change ride height by changing pressure. Recalculate after adjusting.
- **CG height is a manual estimate** — unlike CG fore-aft (auto-calculated from corner weights), CG height has no simple scale trick and stays whatever you've entered in Vehicle Profile until you measure it some other way.
- **Steering Geometry's own pivot geometry is independent** — its track bar/drag link pivot coordinates are a separate concern from the shared Vehicle Profile and live in their own `steering-geometry-v1` storage key, same as 4-Link Geometry's pivots do for the rear.
- **Ride Height Log is intentionally disconnected from Vehicle Profile** — logged ride heights are a measured quantity that changes over time, not a design input the calculators need, so it keeps its own `rideheightlog-v1` storage key and doesn't read or write Vehicle Profile at all.

The in-app **Getting Started** view documents this in more detail, with first-time walkthroughs for each tool.

## App structure

Everything lives in one `index.html` — markup, styles, and logic — so there's no framework or build step to fight. The layout is a sidebar (`.sidebar`) of tools plus a main content area; each tool is a `<section data-view-panel id="view-...">` that the sidebar shows/hides. To add a new tool:

1. Add a `<section class="app" id="view-<name>" data-view-panel hidden>...</section>` inside `<main class="main">`, alongside `view-4link`.
2. Add a `<button class="nav-item" data-view="view-<name>">...</button>` in the sidebar, and drop the `disabled`/`Soon` styling from its placeholder.
3. Give the new tool's script its own IIFE (see the 4-link tool's `<script>` block for the pattern) so its variables don't collide with other tools' scripts.
4. If the tool needs any of wheelbase, tire radius, track width, CG height, or corner weights, read them from `localStorage["vehicleProfile-v1"]` (see the small `loadVehicleProfile()` helper duplicated in the 4-Link, Spring & Bag Rate, and Steering Geometry scripts) rather than adding another copy of those fields — and re-read them on the section's `viewshown` event (dispatched by the nav-switcher whenever that view is shown) so the tool picks up edits made elsewhere in Vehicle Profile.

The 4-link tool's geometry math (four-bar linkage solve, anti-squat calculation), the spring rate tool's math (wheel rate, ride frequency, adiabatic air spring rate), and the steering tool's math (the same four-bar linkage solve, reused front-view, plus the bump steer arc comparison) each live in their own `<script>` block near the bottom of `index.html`, with comments marking each section. The steering tool's `circleIntersect`/`closerTo`/`dist`/`angleDeg` helpers are deliberately copied from the 4-Link script rather than shared, matching the existing convention of keeping each tool's script self-contained.

## Known limitations

The 4-Link and Steering Geometry tools are both 2D approximations (side-view and front-view respectively), and the Spring & Bag Rate tool models a spring/bag in isolation (see each tool's writeup above for specifics). Treat every calculator here as a way to compare designs and sanity-check geometry before cutting metal, not a replacement for measuring or driving the real truck.
