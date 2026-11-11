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

## Rules baked into the form
- Everything converts to **mm** — Mako works in mm, always.
- Mexican stock defaults: sheets **1220 × 2440 mm**, kerf **3 mm**, carcass
  **16 mm melamine** unless changed.
- Missing numbers are flagged as questions. Mako **never invents a
  dimension**.
