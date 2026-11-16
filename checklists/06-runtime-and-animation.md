# 06 — Runtime attachments and animation

## Runtime attachments ([docs/11](../docs/11-runtime-attachments.md))
- [ ] Native mesh's attachment list decoded **as data** by the extractor (user-data classes/values, physics asset, effect mesh).
- [ ] Donor set chosen deliberately (same character's native references = simplest).
- [ ] Condition mesh = owned duplicate; self-link verified to be the condition mesh, not the outfit mesh.
- [ ] Native references declared only as empty class-correct assets at native paths; **excluded from every whitelist** (P21).
- [ ] Protected properties set via console SET and **read back**.
- [ ] Independent inspection of saved **and** cooked: each reference resolves to a real game object (path **and** class); imported exactly once; exactly one KBD and one condition user-data export; condition mesh bound to native skeleton.
- [ ] Saved-bind audit re-run after this stage.
- [ ] Claim limited: "restored links", not "tuned physics".

## Animation (optional; skip if native motion suffices)
- [ ] Decision recorded: why native animations are insufficient.
- [ ] Only exact native paths replaced; combat/shared assets untouched; `combat_assets_modified: false`.
- [ ] Native clip timing, root motion, notifies (every property), curves, flags captured **before** import.
- [ ] Blender: native-bone-only temp armature; action slot resolved; per-frame rebase; root-motion copied from native.
- [ ] Animation FBX = curves only; one clip per export into a fresh directory.
- [ ] Import into a new project; target must not pre-exist; native baselines staged from a separate package root; **no duplicate/reimport of cooked native sequences** (P6).
- [ ] Notifies restored via console SET text; compression/lock/root flags restored.
- [ ] Tolerances met: frames exact; length 1e-5; curves 1e-7; notify times 1e-7; root translation/scale 1e-4; root rotation |dot−1| ≤ 1e-5.
- [ ] Semantic notify comparison PASS (no `failures`).
- [ ] After cook: separate-process reload compared to semantic snapshot.
- [ ] Reuse of verified cooked clips: report triple agrees (cook, semantic, reload); hashes of `.uasset` and `.uexp` equal in all; reuse proof written; no mixing with fresh cooking.
- [ ] Coverage statement written (clips covered; combat, interactions, cutscenes, transitions, face: not covered).
