# Mako — Carpentry Project Requests

The intake form for Kevin's carpentry pipeline. Fill it in, and Mako (the
carpentry CAD specialist agent) gets everything she needs to build the job in
FreeCAD, render it in Blender **with the measures on it**, and produce the cut
list, sheet nesting and materials takeoff.

**Live form:** https://ryuko2.github.io/mako-carpentry-requests/

## How a request reaches Ryuko + Mako
1. Fill the form (a live dimensioned drawing updates as you type).
2. Press **Send as GitHub Issue** — it opens a pre-filled Issue in this repo
   with the full order sheet and `params.json`. Create the issue.
3. Tell Ryuko in chat: "I sent a request." She picks the issue up and runs it
   with Mako through the pipeline.

The form also offers the same request as downloads (`params.json`,
`00-order.md`) or copy-paste text for chat/WhatsApp.

## Cloud features (Firebase)

The form doubles as a small SaaS for managing your own requests, backed by
Firebase (Auth + Firestore), loaded from the page itself — no build step.

**Layout (v4, Lux professional remake):** a sticky app-bar carries the brand
(framing-square mark + MAKO wordmark) and the auth pill — your account state
is always visible at the top. Below it, the **History band** spans the full
width; below that, the two-column workspace: the numbered form (01–08) on the
left, and on the right the **Send panel first**, then the live drawing and the
`params.json` / order-sheet code panels. A product footer closes the page.
Single file, system font stack, inline SVG icons, one spacing/type/radius
token system throughout.

- **Sign in with Google** — the pill in the app-bar at the top of the page.
  Signing in unlocks the cloud save and your history; the form works fully
  offline without it.
- **Save request to cloud** — the primary button in the send panel at the
  **top of the right-hand column** (above the live drawing). It shows a
  loading state while the write is in flight. Writes the request (full
  `params`, the generated order sheet, job name, project type, client, a
  one-line summary and a timestamp) to Firestore as one document, with status
  starting at `new`.
- **My requests · History** — a live-updating list of every saved request,
  newest first, in its own full-width panel **directly under the app-bar**,
  above the form: date, job name, type and overall dimensions. The list has
  all four states: skeleton rows while loading, a designed empty state with a
  "Start a request" call-to-action, an error state that says what happened,
  how to fix it, and offers Retry — and the populated list. Per request you
  can:
  - change its **status** — `new` / `in-progress` / `done` (the pill is
    colour-coded per status);
  - **View** the full stored order text and `params.json`;
  - **Load into form** — pushes the stored params back into the form so you
    can edit and re-save;
  - **Delete** it (a styled confirmation modal names the request; "Keep it"
    is the safe default — nothing is deleted until you confirm).
- **Data model** — one Firestore collection: **`requests`**. Documents carry
  `createdAt`, `status`, `jobName`, `projectType`, `client`, `summary`,
  `params` and `orderMd`.
- **Access** — Firestore security rules restrict the `requests` collection to
  the owner account (Kevin's Google sign-in); nobody else can read or write
  it. The Firebase web config is embedded in `index.html` (a public
  identifier — the rules are what protect the data). If the config were ever
  removed, the page degrades gracefully: the cloud UI shows "not configured
  yet" while every offline feature keeps working.

## Rules baked into the form
- Everything converts to **mm** — Mako works in mm, always.
- Mexican stock defaults: sheets **1220 × 2440 mm**, kerf **3 mm**, carcass
  **16 mm melamine** unless changed.
- Missing numbers are flagged as questions. Mako **never invents a
  dimension**.

## Changelog

### v7.1 — 2026-11-13 (Lux — Eve HOLD fix: EVE-011)

- **Undo is scoped to its request.** The preset Undo snapshot no longer survives loading a different request: `loadParams` (the single funnel for History open / cloud reopen) clears it alongside the design field, exactly as the example loader already did. Before this fix, applying a design on one request and then opening another left the first request's Undo armed — clicking it would silently replace the opened request's state (and a cloud save would have written the wrong content over its record). Within a request nothing changes: apply → Undo still restores the pre-apply layout byte-for-byte.

### v7 — 2026-11-13 (Lux — the design gallery: catalogue presets + Vary)

- **Start from a design.** The Layout step now opens with a gallery of **19 catalogue presets** — 12 closets (The Balanced Three, Max Hang, Esencial, The Dresser Wall, Shoe Gallery, Para Dos, Long-Hang Gallery, Walk-In L, Walk-In U, His & Hers U, Vestidor Abierto, Corredizo Urbano) and 7 kitchens (One-Wall Flow, Pantry Bookend, L Classic, Cook's Triangle L, Galley Wet Side, U Family Kitchen, U Chef) — researched from IKEA PAX/METOD, California Closets, ClosetMaid/Elfa kits, Home Depot and Mexican a-medida shops (`preset-catalog.md`). Each card shows a mini elevation drawn from its own recipe in its own skin, its shapes and size range; the gallery filters by project type, badges presets that will switch the layout shape, and flags ones too small for the piece on screen.
- **One tap applies the whole layout.** Style + materials from the preset's pairing, shape set, and every run rebuilt from the recipe: section percentages convert to millimetres on the run's real inside-width basis (Run 1 = width − carcass; side runs = clear length past the corner), largest-remainder rounding so widths sum exactly, applied as custom widths Kevin can then nudge. Room, overall size and hardware mechanisms are untouched. The order sheet gains a `Design: <name>` line and params an additive `design` field (null for hand-built layouts — every v6.1/v5.1/v4 save still loads losslessly).
- **Too small squeezes, never amputates.** Below a design's sensible size the sections scale proportionally (250 mm floor per section), nothing is dropped or reordered, and a warning through the existing channel names the design and the shortfall; height and side-run minimums warn the same way.
- **Non-destructive.** Applying snapshots the current layout first — **Undo** in the gallery bar restores it exactly. Preset side runs are real runs: the shape-switch stash keeps them across Straight ↔ L/U switches.
- **Vary this design.** One click spins a fresh, still-legal take on the applied (or structurally matched) design: a skin swap from the project's pairing pool, a mirror of the section order (**closets only — kitchen order is never shuffled**), and a density jitter (drawer/shelf counts ±1 within their legal ranges, hinged-pair ↔ sliding rotation on sections ≥600 mm). The order sheet notes it: `Design: Para Dos (variation 2)`.
- **K5 Galley Wet Side** carries its note on the card and after applying: a true galley is two facing runs — this preset is the complete wet side; model the facing run as a second piece.

### v6.1 — 2026-11-13 (Lux — Eve HOLD fix: EVE-005 … EVE-010)

- **3D renders the actual piece.** The back panel is the real back-panel thickness at the back of the carcass (it was an unscaled 4 m slab), and the box builder's minimum-size clamp is at scene scale (it was clamping every panel, shelf and drawer front to a minimum of 1 m per dimension).
- **3D side runs sit in the room, facing the room.** The left run is depth-flipped inside its rotation (its correct placement is a mirror of Run 1's frame — no pure rotation exists); the right run takes the rotation that keeps it inside the room. Backs against their walls, doors on the room-side plane, sections in order from the corner.
- **Loads are faithful in the side runs.** Loading a second L/U request reapplies its side-run sections even when the shape doesn't change (previously the first request's side run survived, and a save would have cemented the mix-up).
- **Shape switches don't eat work.** Switching L/U → Straight → back restores the side runs exactly as entered, from a session snapshot (refreshed on every load; nothing is discarded, so no confirmation is needed).
- **3D first paint.** The preview no longer claims the library "could not load" while it is still on its way: it shows a loading note, builds the moment the library lands, and only reports failure once the load has actually failed.
- **Warnings use the clear length.** Side-run section widths are validated against the run's clear inside width (length − depth − panels), the same basis the drawings build on — an overflowing side run now warns instead of silently spilling.

### v6 — 2026-11-13 (Lux — modular layouts, door mechanisms, interactive 3D)

- **Layout shapes + runs.** The Layout step now opens with a shape picker: Straight (one run, the v5 behaviour), **L** (main run + left side run) and **U** (main + both side runs). Each run sits on its wall with its own length and its own sections list; side runs get a per-run panel with length / divide / count controls. **Corner rule, stated in the form, the plan and the order sheet: the corner bay belongs to Run 1 (main wall) and is counted once — a side run butts into it, so its clear length is its length minus the piece depth.** The room plan draws the true L/U footprint with run labels; a run switcher above the layout elevation picks which run's front is shown (side-run elevations draw the corner bay as a stub owned by Run 1).
- **Division control.** Per run: an explicit **number of sections** (type the count, press *Set count*) and **how to divide** — *Equal widths* (auto-split; widths sum exactly to the run's inside width) or *Custom* (per-section widths, blanks split the remainder, as before). The order sheet prints each run's divide mode and every section's clear width.
- **Drawer counts.** Drawer-bank sections (Drawers only / Shelf tower + drawers) carry an explicit **drawer count, 2–8**, used identically by the params, the order sheet, the 2D layout preview and the 3D view — no more derived default counts. Multiple drawer banks per run are supported.
- **Door types + mechanisms (herrería by name).** Per section, a **door type** separate from door *style*: none/open, hinged single, hinged pair, sliding (bypass), bifold. The 2D layout preview draws them distinctly (swing arcs + hinge ticks, sliding overlap + arrows, bifold fold lines); 3D draws them too. Piece-level mechanisms are now pickers in Hardware and land in the order sheet by name: **hinge type** (European concealed / European soft-close / exposed butt), **sliding system** (top-hung bypass / bottom-rolling bypass), **drawer slide type** (side-mount ball-bearing / undermount soft-close / push-to-open / wooden runners) + slide length. New warnings (warn, never block): sliding doors on sections under ~600 mm, bifold on sections over ~1200 mm, side runs not longer than the piece depth.
- **Interactive 3D preview.** A new step renders the whole piece — L/U shapes assembled, materials as colours, doors per type, drawers, rods and shelves — from the same `gather()` params on the same refresh cycle. Drag to orbit, scroll/pinch to zoom, right-drag / two-finger drag to pan, plus an **Open doors & drawers** toggle and a view reset. Three.js comes from a CDN with the Firebase-style doctrine: if the CDN is unreachable the step shows a short note and every 2D preview keeps working with zero errors.
- **Compatibility.** Params remain a superset: `layout.shape`, `layout.runs[]` (with `layout.sections` still mirroring Run 1 for the existing pipeline), per-section `door_type` + `drawer_count`, and the hardware mechanism fields are all additive; `doors` is still emitted, computed from `door_type`. Every v5.1/v4 saved request loads losslessly (old saves land on Run 1, shape Straight; the old free-text hinge value parks as a stored custom, per the v5.1 preservation mechanism, which also covers every new select). The per-section door-*count* select is gone — superseded by door type.

### v5.1 — 2026-11-13 (Lux — Eve HOLD fix: EVE-002 / EVE-003 / EVE-004)

- **Round-trip preservation for select-bound fields.** A `<select>` silently drops a stored value that is not among its options, so loading a saved request whose door material or style sat outside the pickers' catalogues lost that value on the next save (Eve HOLD, EVE-002 S2 + EVE-003 S3). Stored values outside an option set are now parked in a visible "<value> — stored custom" option and ride through collect unchanged until Kevin deliberately picks something else. Applied to: door material picker, style picker, project type, wall, construction, the three thickness selects, and per-section type / doors / door-style selects. Params shape unchanged; no new dependencies.

### v5 — 2026-11-12 (Lux — per-step live previews, rendered materials, styles)

- **A live preview at the end of every build step**, all inline SVG, all driven by `gather()` on the existing refresh cycle; the sticky column's overall front elevation is unchanged:
  - **Room** → top-down plan: room rect + dims, ceiling noted, chosen wall in amber, piece footprint on that wall (respects centred-on-wall), obstacles drawn at their position with labels.
  - **Piece** → the front elevation + depth (its home step now).
  - **Materials** → construction detail: door/carcass/back thickness bars to relative scale with edge-band indicators, plus the stock sheet with the piece front overlaid to the same scale.
  - **Layout** → front elevation with real internals per section (hanging rods + clothes lines, shelf boards, drawer fronts, kitchen section types too), section widths from the same `bayWidths()` math.
  - **Hardware** → door front with its handle(s), closet-rod end view at Ø scale, drawer-slide side strip at length scale.
- **Materials picker (Materials step)** — curated Home Depot México sheet goods with colour swatches: melamine in 9 shades (Blanco, Blanco Polar, Gris, Negro, Roble, Nogal, Encino, Cerezo, Wengué), triplay pine face + oak/encino face, MDF natural, aglomerado natural, solid pine. Carcass and doors/fronts pick separately; the picks paint every render (flat colour for melamine, procedural SVG wood grain for wood/plywood, layered edge on plywood). The existing free-text name field still works — a custom name renders with a neutral fill. New optional param: `materials.door_material` (`carcass_material` unchanged; the order MD names both).
- **Style picker (Piece step)** — 6 styles with mini door glyphs: Modern / Minimalist, Shaker / Classic, Rustic / Farmhouse, Industrial, Scandinavian, Traditional. One per-style delta table drives door profile, top treatment, base and handles across the renders; picking Shaker presets untouched section door styles to shaker (per-section overrides Kevin made are kept). New optional param: `piece.style`, plus a Style line in the order MD.
- Both params additions are optional and back-compatible — old saved requests and the Mako pipeline read exactly what they read in v4. No new dependencies; still one file, offline except Firebase.

### v4 — 2026-11-12 (Lux, QA'd by Eve — PASS WITH NOTES, EVE-001 fixed + regressed)
The professional remake described under "Cloud features (Firebase)" above: app-bar + auth pill, full-width History band, send panel first in the right column, token-based design system.
