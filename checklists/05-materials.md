# 05 — Materials

## Bake (Blender)
- [ ] Only Principled-BSDF materials accepted; unsupported graphs rejected (human decides), never approximated.
- [ ] Bake UV non-overlapping and inside 0..1; original UV layers byte-preserved.
- [ ] Bakes hash-pinned; non-finite pixels rejected.
- [ ] AO constant 1 verified before using a shared white occlusion texture.
- [ ] Constant colours: sRGB-encoded for the Color parameter (P11).
- [ ] Normal-map convention (green up/down) checked on the first test material.

## Plan translation (no geometry rebaking)
- [ ] Raw bake manifest **rejected** by the editor stage; only the translated plan schema accepted (P10).
- [ ] Parent per alpha mode (opaque vs masked/coverage) = **native** material instance parent; no custom master (P12).
- [ ] Parameter set per material exact; colour-space flags asserted: `srgb == (param == Color)`.
- [ ] Masked: source clip → binary coverage texture; two-sided/alpha recorded.
- [ ] Names unique and within length/charset limits; shared textures only where all sharers agree.

## Editor
- [ ] Plan fully validated **before** creating anything.
- [ ] Native-parent declarations created empty at native paths — and recorded as **not shipped** (P21).
- [ ] Texture overrides written by console SET; `instance.parent` read back after save.
- [ ] **Both** owned meshes (outfit + condition mesh) rebound by slot name; read-back of slot→material map; changed material class ⇒ rebinding both (P9).

## Independent container check (decode actual packed bytes with real native parents)
- [ ] Class = MIC; parent path = plan; each texture parameter exactly once; association/index correct.
- [ ] No `bOverride_*` static overrides true.
- [ ] Mesh chunk decoded == release hash; slot→material map == plan.
- [ ] Bulk files accounted for (P8).
- [ ] Status limited to "parameters verified"; appearance remains NOT OBSERVED until in-game.
