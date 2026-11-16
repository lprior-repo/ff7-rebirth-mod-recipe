# 09 — Native-parent material instances (the materials that actually render)

**Rule: ship material *instances* whose parent is the game's own shader material. Never ship a replacement master material.** [DECISION, after an OBSERVED failure]

## Why [OBSERVED]

A first full candidate used owned plain materials built in the editor (surrogate shader). It cooked, packaged, passed inspection, loaded in the game and moved, but rendered **gray / checkered** — the engine's "no valid shader" look. Fixing the packaging of texture bulk data did **not** change that (docs/13, P8), so the cause was the material model, not missing bytes. Replacing every material with a `MaterialInstanceConstant` (MIC) whose *parent* is a real shader instance from the game (opaque and masked/coverage variants) rendered lit, textured materials in the first in-game test of that candidate. [INFERENCE: the game's shader/permutation set is precompiled for native parents; a new UMaterial has no matching compiled permutations in the shipped game.]

## Plan translation (Blender bake → native parameters)

Run in Blender (needs `bpy`), headless, **no geometry rebaking**. Input: the owned PBR manifest from docs/08. Output: a *translated plan* (new schema id) that is the only manifest the editor stage accepts. Reject the raw bake manifest in the editor stage (explicit schema check) so the wrong file cannot slip through. [DECISION]

Per material, pick the parent by alpha mode:

| Source alpha mode | Parent kind (object path *shape*) |
|---|---|
| OPAQUE | native "Surface standard wide" instance under the game's `Renderer/MaterialInstance/Surface/` |
| MASKED | native "Surface standard coverage wide" instance, same folder |

[OBSERVED] Names used in the tested build were `RMI_Surface_Standard_Wide` and `RMI_Surface_Standard_Coverage_Wide`; find the equivalents for any other build by decoding a native mesh's material chain with the extractor (docs/03) — do not guess.

Texture parameters supplied per material (parameter names as decoded from native instances): `Color`, `Normal`, `WideOcclusion`, `Roughness`, `Metallic`, plus `Coverage` for masked. [OBSERVED]

| Parameter | Source | sRGB flag | Compression |
|---|---|---|---|
| Color | BaseColor bake (or 1×1 constant) | **true** | default |
| Normal | Normal bake | false | TC_NORMALMAP |
| Roughness | ORM **G** channel expanded to gray RGB | false | TC_MASKS |
| Metallic | ORM **B** channel expanded to gray RGB | false | TC_MASKS |
| WideOcclusion | shared 1×1 white (AO was constant 1) | false | TC_MASKS |
| Coverage (masked only) | `1.0 if opacity >= clip else 0.0`, gray RGB | false | TC_MASKS |

Everything except Color must have `srgb = false`; assert `srgb is (param == "Color")` in the editor stage. [OBSERVED rule]

### Hazards and rules

* **Constant colours must be sRGB-encoded (P11).** Generated image bytes bypass the bake's linear→sRGB conversion. A 1×1 constant Base Color read from linear Principled inputs and written as raw bytes will look dark. Apply the piecewise sRGB encode (`v*12.92` if `v<=0.0031308` else `1.055*v^(1/2.4)-0.055`) to RGB only, never alpha, never scalar data. Write the PNG with the image colour space set to Non-Color so Blender does not convert again. [OBSERVED]
* **Masked cutoff**: the native coverage parent has its own cutoff (about 0.333 in the tested parent); bake the coverage texture to binary using the *source* clip so both agree. Record source clip, two-sided flag and alpha mode in the plan; two-sided/backface appearance is **not** implemented as static permutation changes and stays an in-game art gate. [OBSERVED limit]
* Hash-check every retained bake before reading it; reject non-finite pixels; reject AO that is not constant 1 (otherwise the white-AO shortcut is wrong). [OBSERVED]
* Material/texture identities: material path `/Game/<Mod>/Materials/M_<name>`, textures `/Game/<Mod>/Textures/<stem>`; names `[A-Za-z0-9_]`, ≤48 / ≤64 chars; refuse duplicates. [OBSERVED]
* **Shared textures** across materials are allowed only if every sharer agrees on parameter+entry; otherwise fail. [OBSERVED]
* Expect lots of bulk: a few hundred packages and tens of bulk chunks for a full outfit (per-material texture sets dominate). [OBSERVED]

## Editor stage (unreal python, matched editor)

1. Refuse to run unless engine version starts with the matched `4.26.2-`.
2. Validate the plan fully **before** creating anything: schema id, UV index 0, parent matches alpha mode, expected parameter set, colour-space flags, every converted (Windows-path) PNG exists with the recorded hash, whitelist sets equal the union of material + texture packages.
3. Create **native-parent declarations** for each distinct parent: a *parentless* `MaterialInstanceConstant` created at the native object path. It exists only so the editor can resolve the reference; refuse if an asset already exists there (the project must not already contain a copy of the native one). These are **never shipped** (docs/13 exclusion list).
4. Import textures with an automated `AssetImportTask` (`replace_existing=False`), set `srgb` and compression per the table, save.
5. Create each owned MIC at its path, set its parent via the material editing library, then write the texture overrides with a console `SET <instance> TextureParameterValues (...)` using `ParameterInfo=(Name="<param>",Association=GlobalParameter,Index=-1)` and `ParameterValue=Texture2D'<path>'`. [OBSERVED] Reason: the library setter rejects parameters that are absent from the intentionally empty parent declaration; the property import-text route persists them regardless.
6. Save, then confirm the parent reference survived (`instance.parent == declaration`).
7. **Bind both owned meshes** (outfit mesh and the condition mesh, docs/11) — match slots by *imported* slot name first, then slot name; fail if any visible slot has no material or the counts differ. Save and read back the slot→material paths. [OBSERVED]
8. **Changing material class at an existing path requires rebinding both meshes**: old UMaterial imports do not resolve as new MIC objects. (P9)

## Independent actual-container inspection (do not skip)

Decode the *packed* containers + the *real* game containers together (so native parents resolve) in an out-of-editor reader (docs/03). Assert for every material:

* class is the MIC class; parent object path equals the plan's native parent;
* `TextureParameterValues` entries: each name exactly once, association `GlobalParameter`, index −1, and object paths equal the plan;
* no `bOverride_*` base-property flag is true (no static shader property overrides);
* the mesh chunk the reader decoded has the same hash as the one in your release;
* slot names → material paths equal the plan.

Record `status: PASS_...` plus hashes. This proves **parameters**, not UVs, not appearance. [OBSERVED limit]

## Reuse of an accepted outfit when only materials change

Provide a phase that: copies every non-material, non-texture file from the accepted candidate's project (hash-verified), excludes the native-path declarations, re-copies prior reports with an "accepted_source_reuse" mark, creates fresh material/texture assets from the new plan, rebinds both meshes, and writes a provenance record saying geometry was not reimported and textures were not rebaked. [DECISION] This saves a 10+ minute editor import and avoids re-triggering bind/animation gates.
