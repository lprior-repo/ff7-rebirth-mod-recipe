# 04 — FBX export/patch, editor import, saved-bind audit

## FBX
- [ ] Export settings pinned in config and match [docs/07](../docs/07-fbx-native-bind.md) (axes, no space-transform bake, no leaf bones, rest pose, rig object name).
- [ ] FBX written once; patched **copy** produced with a report (never patch in place).
- [ ] Post-patch preconditions: mesh world == identity, bone parents == authority, unique model names.
- [ ] Patch report has payload-unchanged digest for nodes not meant to change.
- [ ] Re-parsed FBX compared to authority: local transforms within 1e-4 abs / 1e-5 rel; quaternions normalised; scale > 0. Values recorded.
- [ ] Source scene hash unchanged.

## Editor import (fresh candidate project)
- [ ] Engine version string starts with the matched `4.26.2-`; otherwise refuse.
- [ ] Import options pinned per the table in [docs/10](../docs/10-editor-import-and-saved-bind-audit.md) (native skeleton, no new skeleton, no physics asset, no materials/textures, **do not update skeleton reference pose**).
- [ ] Host-side timeout; display available; CPU throttled.
- [ ] Report JSON exists, status matches, no `error`; log scanned for **bind-pose / relative-transform warnings** — any is a FAIL until explained and fixed (P3).
- [ ] Native shared skeleton on disk is hash-identical after the run; never saved, never shipped.

## Saved-bind audit
- [ ] Saved mesh contains all expected bones with correct parents.
- [ ] Local **and** world bone deltas vs authority ≈ float noise (record max).
- [ ] Material slots present with expected names/count; sections/vertices counts recorded; LOD count recorded.
- [ ] Re-run after any later stage that re-saves the mesh (runtime attachments, material rebinding).

## Pass
- [ ] Report path + hashes in ledger; unobserved aspects (UV look, shading) stay NOT OBSERVED.
