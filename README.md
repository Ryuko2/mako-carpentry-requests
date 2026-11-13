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
