# 03 — Blender revision and game-source prep QA

Mandatory before touching Blender: load whatever Blender skills the environment provides, **including in delegated work**; apply their procedures (not just the names). Scene-specific invariants (native-cm rig, contract) override generic tutorial defaults.

## Revision hygiene
- [ ] New `.blend` per revision, named by purpose + counter, derived from a hash-recorded parent; parent never edited.
- [ ] Script is single-purpose, headless, writes into a fresh dir, emits a JSON report (input hashes, counts, measured values, failures).
- [ ] **Saved → reopened → measured**: QA ran on the reopened file.
- [ ] Untouched meshes' topology/UVs/weights byte-compared to the parent: "preserved" with evidence.

## Geometry QA (reopened scene)
- [ ] No non-finite coordinates; no loose junk; manifold/hole review for body-visible shells.
- [ ] Garment ↔ body clearance (KD-tree/BVH): min/max recorded; poke-through count.
- [ ] No coplanar/z-fighting pairs; terminal-edge gaps to native meshes measured.
- [ ] UVs: original layers preserved; any new layer non-overlapping inside 0..1; texel density sane.
- [ ] Symmetry/profile errors within the human's tolerance (record mean/max).
- [ ] Polygon/vertex budget reported vs stock (stock LOD0 is the yardstick).

## Rig and skin contract
- [ ] Exactly one armature; bone-name set == authoritative set; every parent == authoritative parent.
- [ ] Every visible mesh bound to that armature; vertex groups named as bones.
- [ ] No unweighted vertices; weights normalised; **≤ 8 influences** per vertex (report count and weight mass that any cap would remove).
- [ ] Linear blend skinning only (no dual quaternion).
- [ ] Rig pose set to rest, actions cleared, NLA muted; unit scale 0.01; scene in native cm.
- [ ] Presentation offsets (floor lift, stage parents) removed; height policy recorded as a runtime requirement.
- [ ] Facial/deformation probes (bounded, exact bone transform → per-material displacement → pose restored exactly → input hash unchanged) for face and any problem region.

## Slots and modifiers
- [ ] Surface modifiers applied (Armature kept); viewport == render visibility.
- [ ] Material slots compacted to used slots, order preserved, names preserved; no empty used slot.

## Pass
- [ ] Prepared `.blend` + manifest in fresh dir; **source hash unchanged** before/after.
- [ ] Status vocabulary used; `PREPARED_NOT_COOKED`-style status (not "ready").
