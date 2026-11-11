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
