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

- **Sign in with Google** — the auth bar at the top of the page. Signing in
  unlocks the cloud save and your history; the form works fully offline
  without it.
- **Save request to cloud** — next to *Send as GitHub Issue*, in the send
  panel at the **top of the right-hand column** (above the live drawing).
  Writes the request (full `params`, the generated order sheet, job name,
  project type, client, a one-line summary and a timestamp) to Firestore as
  one document, with status starting at `new`.
- **My requests · History** — a live-updating list of every saved request,
  newest first, in its own full-width panel **directly under the page
  header**, above the form: date, job name, type and overall dimensions.
  Per request you can:
  - change its **status** — `new` / `in-progress` / `done`;
  - **View** the full stored order text and `params.json`;
  - **Load into form** — pushes the stored params back into the form so you
    can edit and re-save;
  - **Delete** it (with a confirmation).
- **Data model** — one Firestore collection: **`requests`**. Documents carry
  `createdAt`, `status`, `jobName`, `projectType`, `client`, `summary`,
  `params` and `orderMd`.
- **Access** — Firestore security rules restrict the `requests` collection to
  the owner account (Kevin's Google sign-in); nobody else can read or write
  it. The Firebase web config lives in `index.html` (placeholder
  `/*__FIREBASE_CONFIG__*/`, filled in when the project is provisioned); until
  then the page degrades gracefully and the cloud UI shows "not configured
  yet" while every offline feature keeps working.

## Rules baked into the form
- Everything converts to **mm** — Mako works in mm, always.
- Mexican stock defaults: sheets **1220 × 2440 mm**, kerf **3 mm**, carcass
  **16 mm melamine** unless changed.
- Missing numbers are flagged as questions. Mako **never invents a
  dimension**.
