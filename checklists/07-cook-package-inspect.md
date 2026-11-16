# 07 — Cook, package, inspect

## Before cook
- [ ] Saved-bind audit, runtime inspection (if used) and material plan are PASS for **this** candidate.
- [ ] Whitelist is an explicit sorted list from import reports; does **not** intersect `native_references_not_shipped`, the native skeleton, or native parent materials.
- [ ] Candidate directory is fresh; parent reuse (if any) has a hash-proof.

## Cook
- [ ] Single-package-no-refs mode (so the closure is not dragged in).
- [ ] Expected set == actual set; missing **and** unexpected both FAIL; exit code 0 not trusted (P1).
- [ ] Bulk files (`.ubulk`/`.m.ubulk`/`.uptnl`) listed with size and hash; every bulk file maps to a whitelisted package.

## Package
- [ ] Response file: UTF-8 with BOM, sorted; suffix set correct; **`.uexp` not listed** (P7).
- [ ] IoStore run into a fresh dir; `.utoc` and `.ucas` both present; one container stem `<Mod>_<candidate>_P`.
- [ ] Companion provenance pak created and listed.
- [ ] `release.json` written: stage `COOKED`, game build id, candidate id, source hash, files (relative, canonical), `gameplay_verified: false`.
- [ ] Package report status `PENDING_INDEPENDENT_INSPECTION` with an explicit `excluded` list.

## Independent inspection (different tool from the producer)
- [ ] Release file hashes == manifest; exactly one container + companion; nothing extra.
- [ ] Container header/entries decode; package IDs unique; chunk table lists the bulk chunks.
- [ ] Every expected package present; no native shipped asset present (skeleton, parents, declarations).
- [ ] Mesh: bones/slots/LOD/section counts match saved report; runtime attachments (if used) resolve; animation bytes unchanged vs verified source (if reused).
- [ ] Materials: container parameters match plan (checklist 05).
- [ ] Texture pixels decode (presence only — not appearance).
- [ ] Final status carries its limit: `…_CONTAINER_STRUCTURE_NOT_RUNTIME`.

## Hand-off
- [ ] Ledger row updated from the report (path + hash), not from memory.
