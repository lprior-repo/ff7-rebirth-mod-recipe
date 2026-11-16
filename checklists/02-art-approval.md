# 02 — Art approval (before any editor/cook/install work on a scene)

## Base and references
- [ ] Base model chosen from a rubric: anatomy, UV quality, weights, polygon budget, **licence/permission to modify and use** (read the author's terms; prohibited bases never enter the pipeline).
- [ ] One **main target** reference image designated; others supplement. Location recorded privately (not committed unless the human says).
- [ ] The frozen **contract** written down: native-cm units, native rest rig, bone set/names, UV layer names, vertex-group names, material-slot names.

## Review renders (cheap, fixed)
- [ ] Fixed camera set matching reference framing: front, 3/4 left, 3/4 right, side, back, 1–2 close-ups; same set every revision.
- [ ] Cycles 24–32 samples, ~900 px wide; clay + textured pass.
- [ ] One **large labelled image per view** (no thumbnail strips).
- [ ] Renders were **looked at** (not just produced); side-by-side vs the main reference; differences listed.
- [ ] Numeric aids captured where relevant: symmetry error, profile error vs reference contour, garment-body clearance min/max.

## Approval
- [ ] Human approval given in words, dated.
- [ ] Approval recorded as: file path + hash of the **exact `.blend`**, which views were judged, what it **is** ("art for game test") and what it is **not** ("not bind/physics/gameplay").
- [ ] No pipeline work started on any scene without such a record.
- [ ] Previously rejected scenes/deployments stay rejected unless the human says otherwise in writing.
- [ ] If the human edits the approval scope, the ledger gets a new dated entry; old entry kept.

## After approval
- [ ] Approved scene copied/derived by hash into the game-source prep input; the approved file never modified.
