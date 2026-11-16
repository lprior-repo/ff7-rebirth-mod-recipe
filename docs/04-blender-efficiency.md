# 04 — Blender authoring: being efficient, lessons learned

The source project produced ~60 numbered art revisions, five-view renders, and a rejected deployment. This doc captures how to avoid that cost. Tags: `[OBSERVED]` happened; `[GENERAL]` standard practice.

## 1. Order of work

1. **Prove the pipeline on a trivial mesh first** (docs/17 S1). A recolor, a tiny geometry tweak, or a straight native round-trip proves extraction→Blender→FBX→editor→cook→install→load with almost no art risk. Do this before sculpting anything. [DECISION]
2. **Get art approval on cheap evidence** (low-sample Cycles, 800–900 px, 3–5 camera angles, clay + textured) *before* running the heavy technical pipeline on a scene. The source project ran the whole pipeline on a model and the human rejected the in-game look afterwards. [OBSERVED]
3. **Freeze a contract** so any approved scene can flow through unchanged: native-unit scale, native rest-pose rig, bone set/names, UV layer names, vertex-group names, slot names. Art work may change shapes; it must not change the contract. [DECISION]
4. **One immutable `.blend` per revision**, derived from an approved parent by script (or hand), with the parent's hash recorded. Never edit an approved file. Name by purpose + counter. [OBSERVED]

## 2. Efficient scripted Blender work

* Run **headless** (`blender -b scene.blend --python script.py -- args`). Parse args after `--`. Fail fast with explicit exceptions, not warnings. [GENERAL]
* Keep each script single-purpose and **idempotent into a fresh output dir**; write a JSON report with: input hashes, counts, measured numbers, pass/fail lists, and "did not modify source".
* **Save then reopen then measure.** Many defects (shared data, lost modifiers, lost custom properties) only show after reopening. QA must run on the reopened file. [OBSERVED]
* Use spatial structures for cheap geometric checks: KD-tree/BVH for clearance (garment vs body), coplanar-pair/z-fighting detection, terminal-edge gap to a native mesh. Report min/max values and counts, not just pass/fail. [OBSERVED metrics: min body clearance cm, max vertex gap cm, zero coplanar pairs]
* Bilateral symmetry error and profile-sample error against a reference contour are good numeric gates for body shapes (mean/max cm, left-right diff). [OBSERVED]
* **Bounded synthetic probes** for facial/deformation: pose a bone by an exact transform, measure per-material vertex displacement, confirm pose restored exactly afterward, input hash unchanged. Cheaper and more honest than eyeballing. [OBSERVED]
* Only change what the revision targets: topology/UVs/weights of untouched meshes must be byte-compared before/after. Report "preserved" with evidence.
* Rendering for review: Cycles 24–32 samples, ~900×1350 or 900×1100, fixed camera set (front, 3/4, side, back, close-ups). Cheap enough to run every revision. Inspect the actual images; a render script finishing is not review. [OBSERVED settings]

## 3. Native-unit and rig discipline

* The game rig is in **centimetres**. Keep the scene in native cm coordinates; set unit scale 0.01 so export applies the scale correctly. Do not rescale to match tutorials. [DECISION]
* A "presentation" parent (e.g. +8 cm Z lift used to stand the model on a floor for renders) must be **removed before game prep** by re-expressing objects in native coordinates; record the lift as a *runtime* requirement (foot grounding/IK), not something to bake. [OBSERVED]
* Blender 5.x actions use **slots**; scripts that touch actions must resolve the object's action slot. [OBSERVED in animation export]

## 4. Mistakes that cost time (avoid) [OBSERVED]

| Mistake | Cost | Instead |
|---|---|---|
| Starting from a mixed/blocky model then swapping bases repeatedly | many abandoned revisions | choose the base from a rubric (anatomy, UV, weights, licence) before modelling |
| Treating five references as equal | rework | name one **main target** image; others supplement |
| Heavy art before pipeline proof | late discovery of runtime issues | S1 slice |
| Asking the human to approve a scene already in the pipeline | wasted cook/install cycles | approval first, pipeline second |
| Rebaking geometry to diagnose shaders | hours | translate materials separately from geometry bakes |
| Rebuilding when only packaging was wrong | hours | repackage reusing verified cooked bytes (docs/13) |
| Re-rendering with unreadable strip images | confusion | one large image per view, labeled |

## 5. Blender skills routing (if available)

When an agent platform offers Blender specialty skills, load the routing skill and every applicable specialty (scene setup, sculpt, retopo/UV, clothing, hair, rigging, PBR, texturing, lookdev, render, asset QA; plus baking/animation/facial/export when used). Apply their procedures *within* the native contract above. [OBSERVED owner instruction]

## 6. Licences of base assets

Third-party outfit/body bases carry licences. Record author, version, permission and what you may modify/redistribute **outside this repo** (it must not hold them). Never silently mix assets of unknown provenance.
