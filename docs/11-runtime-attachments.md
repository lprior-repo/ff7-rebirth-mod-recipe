# 11 — Runtime attachments: the links a native character mesh carries

A native playable-character `SkeletalMesh` is more than geometry. It carries object references that other game systems (secondary physics, kinematic drivers, effects, the "condition"/outfit-state system) look up at runtime. A freshly imported mesh has **none** of them. [OBSERVED]

## Why this stage exists [OBSERVED / INFERENCE]

* [OBSERVED] Comparing the native mesh to the first full candidate (decoded with the extractor) showed the native mesh had: a **condition-model link**, **three Bonamik (cloth/hair secondary-motion) data references**, **one KineDriver data reference**, a **physics asset**, and an **effect-appendix mesh reference**. The candidate had none. It still loaded, rendered and moved in game.
* [INFERENCE — not proven] These absences plausibly explain missing secondary motion and possibly runtime instability. The source project never proved the missing links caused any crash (two later SIGSEGV exits remained unattributed even *with* the links restored; P17). So: restore them because they are measured differences from native, **not** because a crash was shown to depend on them.

Do this stage after import + saved-bind audit and **before** cook. Re-run the saved-bind audit afterwards (the stage re-saves the mesh; docs/10 §5).

## Recipe (editor Python, `-run=pythonscript`, create-only report)

Precondition: engine version string starts with the expected `4.26.2-`; owned condition-mesh path must **not** already exist (immutability).

1. **Decode the native mesh first** (extractor, docs/03) and write down its attachment list as data: asset-user-data classes, their property values, `PhysicsAsset`, `EffectAppendixMesh`. Do not copy a list from this repo — class and asset names differ per character and build. [GENERAL]
2. **Pick the donor set.** Which native references to reuse is a design choice: reusing the *same character's* native references is the simplest. [OBSERVED] The tested mod reused references from the matching native outfits (three Bonamik instances drawn from related outfits, one KineDriver, a shared humanoid physics asset, the character's effect mesh) rather than authoring new physics.
3. **Duplicate the outfit mesh** to an *owned* package (`<Mod>/Runtime/<Name>Condition`). This is the "condition model": the engine's condition system expects a separate mesh object it can swap to. Both the outfit mesh and the condition mesh receive the attachments.
4. **Declare native references without shipping them.** For each native-path object you only need to *point at*, create an **empty asset of the correct class at the native object path** (`AssetTools.create_asset(name, package, class)`, then save). That gives the editor a resolvable reference to serialise; at cook the reference is stored as an import to the real game object. [OBSERVED] Classes used: `/Script/BonamikRt.SQEX_BonamikAsset`, `/Script/KineDriverRt.SQEX_KineDriverData`, `/Script/Engine.PhysicsAsset`, `/Script/Engine.EffectAppendixMesh`.
   * Refuse if the path already exists in the project (the declaration would collide with something real).
   * **These declarations are never packaged.** If one lands in the container it shadows the real native asset (P21). Record them in the report as `native_references_not_shipped`.
5. **Create the user-data objects on each mesh** (`new_object(cls, outer=mesh, name=...)`):
   * `/Script/KBDRt.SQEX_KBD_AssetUserData` with `BonamikDataList = [..]`, `KineDriverDataList = [..]`.
   * `/Script/EndGame.EndCharacterConditionUserData` with `ConditionModel = <owned condition mesh>`.
6. **Set the properties python cannot reach with the console `SET` command** (`SystemLibrary.execute_console_command`):
   * `SET <mesh path> EffectAppendixMesh EffectAppendixMesh'<effect path>'`
   * `SET <mesh path> AssetUserData (SQEX_KBD_AssetUserData'<kbd path>',EndCharacterConditionUserData'<condition path>')`
   * `physics_asset` is a normal editor property: set it directly.
   The console route is the workable way to set protected/unexposed properties from the editor's Python in this build (same trick as animation notifies, docs/12; P5).
7. Save both meshes; write a report with: both mesh paths, user-data object paths, resolved reference paths read back from the objects, `owned_extra_packages` (the condition mesh), `native_references_not_shipped`, `status: SAVED_..._PENDING_INDEPENDENT_INSPECTION`.

## Independent inspection (do not trust the stage's own report)

Decode, with the extractor (not the editor), both the **saved** and **cooked** (and later the **packed container's**) objects, and assert:

* Each reference resolves to a **real game object**: path *and* class name match what the native mesh references (not just "path exists").
* The cooked mesh imports each native reference **exactly once**.
* The cooked mesh exports **exactly one** `SQEX_KBD_AssetUserData` and **exactly one** `EndCharacterConditionUserData`.
* The condition mesh is a `SkeletalMesh` still bound to the native skeleton.
* None of the declaration paths appear among the cooked/packed packages.

Status vocabulary used: `PASS_SAVED_RUNTIME_ATTACHMENTS_AND_REAL_NATIVE_REFERENCE_IDENTITIES`, then for cooked/container `PASS_COOKED_..._GRAPH`. The **cook stage refuses to run** until the saved inspection passed, and refuses if its whitelist intersects `native_references_not_shipped`. [DECISION]

## What this does *not* give you

Restored links are **native** physics/driver data tuned for the native mesh. They do **not** provide secondary motion for new chest/skirt/hair geometry, nor correct collision for a different silhouette. Treat "restored" and "tuned" as different gates; custom secondary physics was not implemented or accepted in the source project (docs/15 coverage matrix).

## Edge cases

* An asset-user-data property set via `SET` that silently fails leaves the property empty. Always read it back (step 7) and again from the cooked bytes.
* Duplicating the mesh before attaching user data means the condition mesh has a *copy* of the same attachments including a link to itself. Verify the self-link is the intended object, not the outfit mesh.
* Do not add the native physics asset to the whitelist "so it is included": it is a game asset; the reference alone is the point.

Next: [docs/12 animation](12-animation.md) (optional) or [docs/13 cook and package](13-cook-package-inspect.md).
