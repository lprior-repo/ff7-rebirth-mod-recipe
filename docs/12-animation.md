# 12 — Animation replacement (optional stage)

Skip this stage entirely for a first release: a new outfit that binds to the native skeleton plays the **native** animations unchanged. Animation replacement is only needed when the new body's proportions make native motion visibly wrong (e.g. different hip/leg/arm geometry). It is the highest-risk stage; the source project replaced **nine neutral, non-combat clips only** (idle, walk, run, an idle variant, a gesture idle, talk idle, talk gesture, a talk-idle transition, a walk-turn transition) and accepted nothing about combat.

## Rules that survived [DECISION]

1. **Replace at the exact native asset path** (override model). Never create a new animation and hope something references it.
2. **Same skeleton, same bone tracks.** The FBX carries a **subset of the native bone names** (no new bones, no duplicates). If the authoring rig has extra bones, bake onto the native-bone-only armature first.
3. **Do not touch combat/shared assets.** Whitelist exactly the clips you replace. Record `combat_assets_modified: false`.
4. **Preserve everything the native clip carried** that is not bone motion: frame count/length, rate, root-motion flags and lock type, compression settings, curves, and **notifies/events** (footsteps, state changes). A clip that moves right but drops its notifies breaks gameplay silently.

## Blender side (animation FBX) [OBSERVED]

Run in Blender 5.x headless; the source scene is **never saved**; each export comes from a **temporary native-bone-only armature copy** (exact native bone count/order/hierarchy — assert it).

* **Blender 5.x action slots:** actions need a slot bound to the rig; code written against the pre-4.4 `action.fcurves` API fails. Look up the slot, read channels through the channel-bag API, and fail loudly if an action has no object slot. (P23)
* **Capture against native timing:** for each clip load the native sequence's timing (frame count, sample times, rate ≈ native rate within tolerance, here 30 fps) from the extractor, and sample the authoring action at exactly those times. Reject an action that does not span ≥ 2 timestamps, or whose channels name bones outside the native set.
* **Rebase per frame** from authoring rest to native rest: for each native bone compute the pose matrix that reproduces the authored deformation relative to the native rest. Reject singular/non-finite bases at any frame.
* **Root motion:** copy the native root bone's (`Trans`-style) per-frame matrices over the captured root so travel distance/speed matches the native clip. The game's locomotion logic expects the native root motion; a "correct-looking" authoring root that differs breaks foot/travel sync. Apply it to **every** clip and refuse export if any is missing.
* **One clip per export call;** output directory must be fresh/empty. FBX contains **curves only, no geometry**. Status `..._FBX_EXPORTS_READY_NOT_COOKED`.
* Reflections: the authoring scene may be mirrored relative to the engine (handedness). Apply a fixed reflection matrix consistently on capture; verify by comparing a non-trivial bone chain (hand/foot) world position against the native clip at 3+ frames. [GENERAL]

## Capture native events *before* you import

From the pristine native clips, with the extractor, capture each clip's: events (notify trigger time/duration/name/class and **every serialized property**), custom notify/state objects and their properties, curves, root-motion flags, and the property-tag text of the `Notifies` array. Mark undecoded properties as a hard failure (`tagAvailable` all true) — an event with a field you cannot read cannot be restored. Status `PASS_NATIVE_ANIMATION_NOTIFY_CAPTURE`.

## Editor import (per clip, new candidate project) [OBSERVED]

* **Do not duplicate/rename/reimport the cooked native `AnimSequence` assets in the editor.** In the matched editor they are broken for this (P6). Instead: stage the **immutable native baselines** from a separate package root (copied native inputs, hash-pinned), and import **fresh** targets at the exact native paths with `AssetImportTask`.
* `FbxImportUI`: `automated_import_should_detect_type=False`, `mesh_type_to_import=FBXIT_ANIMATION`, `import_as_skeletal=False`, `import_mesh=False`, `import_animations=True`, `skeleton=<native>`, `import_materials=False`, `import_textures=False`. `anim_sequence_import_data`: `use_default_sample_rate=False`, `custom_sample_rate=30`, `preserve_local_transform=True`, `import_bone_tracks=True`, `import_custom_attribute=True`, `delete_existing_custom_attribute_curves=False`, `delete_existing_non_curve_custom_attributes=False`.
* The target must **not** already exist (`replace_existing=False`); the reload mode asserts the opposite.
* After import, restore what the FBX cannot carry, **from the native baseline via `AnimationLibrary`**: `copy_anim_notifies_from_sequence`, root-motion enabled, lock type, "lock forced", bone and curve **compression settings**, `finalize_bone_animation`.
* **Notify event structs cannot be fully copied in Python** (protected fields; P5). Restore by writing the native `Notifies` property text through the console `SET <sequence path> Notifies <text>` after building the text from the captured property tag with the right outer object names. Verify by snapshotting again.
* Save the sequence, never the skeleton. Report `owned_package_whitelist` = exactly the targets.

## Validation after import (before cook) [OBSERVED tolerances]

Snapshot native baseline vs new sequence and require:

| Check | Tolerance |
|---|---|
| Skeleton object path unchanged | exact |
| Frame count | exact |
| Sequence length (s) | `1e-5` |
| Root-motion enabled / lock type / lock forced | exact |
| Curve names + values | equal (≤ `1e-7`) |
| Notify count, order, **trigger times** | exact count; times `1e-7` |
| Root-bone pose per frame — translation and scale | `1e-4` |
| Root-bone pose per frame — rotation | `|dot(q_a, q_b) - 1| ≤ 1e-5` |
| Bone tracks ⊆ native bone names; no duplicates | exact |

Then a **semantic comparison** of notify structs and custom notify UObjects (saved vs native) — every property. Status `PASS_FULL_RUNTIME_NOTIFY_SEMANTICS` per clip; any `failures` list non-empty = FAIL.

## After cook: separate-process reload [OBSERVED]

After cooking, open the **cooked** sequences in a **different editor project/process** and read: root poses, timing, curves, flags. Compare to the semantic snapshot. This separates "the saved asset was right" from "what the cooker emitted". Status `PASS_..._COOKED_ROOT_TIMING_CURVES_FLAGS`. Do not skip: the cooker is allowed to change compression.

## Reuse instead of recook [OBSERVED / DECISION]

Once nine clips passed saved + cooked gates, later model candidates **reuse the cooked bytes** rather than reimporting:

1. Verify the source candidate's cook report, semantic report, and reload report all say PASS, and agree on engine hash, game build id, skeleton path, metadata hash.
2. For each clip, require the cooked `.uasset` **and** `.uexp` hash to equal both the cook report and the reload provenance; require the semantic report was computed on those same bytes.
3. After the new cook, **copy** those files into the new cooked tree (refuse to overwrite), re-verify each hash, add the packages to the expected set, and write an `unchanged-cooked-movement-proof` listing every copied hash and `animations_reimported: false`, `animations_recooked: false`.
4. Never combine fresh animation cooking and verified reuse in one candidate.

## Coverage statement (put this in every report)

"Passes for clips X on skeleton Y; combat, interactions, cutscenes, transitions not listed here, facial animation: **not covered**." Native clips not replaced keep playing on the new body; mixing is where visual pops come from (a replaced idle blended with native transitions). That was not evaluated in game beyond walking and an idle. [OBSERVED limit]

## Efficiency

* Do the Blender capture once per clip set and keep the FBX hashes; the editor import + notify restoration is the heavy step. [INFERENCE: durations were not measured in the source project.]
* Reuse of already-verified cooked clips (above) avoids repeating import, notify restoration, cook and reload verification for every model iteration. [DECISION]
* If the new body shares the native proportions, skip animations and remove a whole stage of risk.

Next: [docs/13 cook, package, inspect](13-cook-package-inspect.md).
