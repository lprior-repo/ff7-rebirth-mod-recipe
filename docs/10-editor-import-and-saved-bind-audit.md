# 10 — Editor import and the saved-bind audit

Goal: turn the prepared FBX into a **saved UE 4.26 `SkeletalMesh` uasset bound to the game's own `Skeleton`**, then prove — from the saved bytes, outside the editor — that the reference pose in the file equals the native one.

Everything in this stage runs inside a matched UE 4.26.2 editor under Proton, headless (docs/02). Only the *audit* runs outside it.

## 1. Isolated project per attempt [DECISION]

* Never import into the shared editor project. Clone it into a **fresh per-candidate project directory** (new candidate id, 8 hex chars; the directory name is the id). Verify the cloned `.uproject` hash against a pinned value, copy the engine config you need, and copy the native inputs the import must see (the protected native `Skeleton` `.uasset`/`.uexp`, compression settings, anything the mesh references) **by hash**, recording each copy `{source, destination, sha256}` in a `candidate.json`.
* `candidate.json` is the state record for every later stage: candidate id, output directory, project path+hash, editor/Proton pins, engine binary path+hash, prepared-source path+hash, authority-metadata path+hash, target mesh asset path, native skeleton object path, game build id, list of copied native inputs, `native_shared_skeleton_shipped: false`, `game_files_modified: false`.
* **Every stage re-verifies these hashes first** (a `context()` step) and again **after** the editor run. If the engine binary, project, source, metadata, copied native input or Proton version changed → stop. [DECISION]
* Reason: one candidate's failed import must not poison the next; a failed directory stays as evidence (docs/01).

## 2. Running the editor [OBSERVED]

One command shape for every editor phase (import, runtime restore, animation, material, cook, IoStore):

```
nice -n 10 [taskset -c <3 cores>] env -u WINEPREFIX -u PYTHONPATH -u PYTHONHOME \
  <proton> run <UE4Editor-Cmd.exe> <project.uproject-as-Z:\path> \
  -run=pythonscript -script=<Z:\path\to\script.py> \
  -NullRHI -unattended -NoSplash -NoSound -NoP4 -NoShaderCompile \
  -stdout -FullStdOutLogOutput -UTF8Output -abslog=<Z:\path\to\Phase.log>
```

Environment: `STEAM_COMPAT_DATA_PATH=<editor prefix parent>`, `STEAM_COMPAT_CLIENT_INSTALL_PATH=<steam client>`, `SteamAppId=0`, `SteamGameId=0`, `WINEDEBUG=fixme-all`, `PROTON_LOG=1`, `PROTON_LOG_DIR=<candidate output>`.

* **Paths** handed to the editor are Wine `Z:` paths with backslashes (`Z:` + host path). Keep one helper for this and use it everywhere.
* **Script I/O** goes through environment variables naming a request JSON (inputs) and a report JSON (output). The editor Python writes the report; the host reads it. Never parse the editor's console log for results.
* **Cap CPU** (`nice`, `taskset` to ≈3 cores): the machine is also running the game/LLMs/desktop. [DECISION]
* **Timeouts** (host-side, kill the process tree on expiry): import 600 s, runtime restore 180 s, cook 1800 s, IoStore 300 s, UnrealPak 120 s. A timeout with **no report and no log** is a display problem until proven otherwise (P13/P14, docs/16).
* The editor needs an X display even with `-NullRHI` under Proton (Wine's window driver). If the host X socket is gone the editor hangs with no output → provision a dedicated authenticated Xwayland on another display number for editor runs only (P14). Do not touch the desktop session.
* **Exit code 0 is not success.** Success = report JSON exists, `status` is the expected string, and no `error` key (docs/01).

## 3. Import settings [OBSERVED]

Import through `unreal.AssetImportTask` (`automated=True`, `replace_existing=False`, `save=False`; save yourself after verification) with a `FbxImportUI`:

| Option | Value | Why |
|---|---|---|
| `automated_import_should_detect_type` | `False` | Force the type; auto-detect picked wrong ones |
| `mesh_type_to_import` | `FBXIT_SKELETAL_MESH`; `import_as_skeletal=True`; `import_mesh=True` | Skeletal mesh |
| `skeleton` | the **native** `Skeleton` asset | Bind to the game skeleton, do not create one |
| `create_physics_asset` | `False` | Runtime physics comes from native references (docs/11) |
| `import_animations` | `False` | Separate stage (docs/12) |
| `import_materials`, `import_textures` | `False` | Owned material instances are created by a separate script (docs/09) |
| `skeletal_mesh_import_data.update_skeleton_reference_pose` | `False` | **Never overwrite the native skeleton's reference pose** |
| `use_t0_as_ref_pose` | `True` | Use the FBX's pose, not an animation frame |
| `import_morph_targets` | `False` (unless a deliberate morph feature) | Less to go wrong |
| `import_mesh_lods` | `False` | Single LOD unless LODs are a planned feature |
| `import_meshes_in_bone_hierarchy` | `True` | Meshes parented under bones in the FBX stay |
| `preserve_smoothing_groups` | `True` | |
| `normal_import_method` | `FBXNIM_IMPORT_NORMALS` | Use authored normals |
| `import_uniform_scale` | `1` | Native units already (docs/05) |
| `convert_scene`, `convert_scene_unit` | `True` | With the FBX exported in the axis/unit the post-patch declares (docs/07) |
| `force_front_x_axis` | `False` | |

Pin this table in a version-controlled config for your mod. A changed option is a changed experiment.

[INFERENCE] `import_mesh_lods=False` + a single LOD is a cost decision, not a proven requirement; whether Rebirth's LOD streaming needs ≥2 LODs was not tested.

## 4. Immediately after import (same editor session) [OBSERVED]

Perform and write into the report:

1. Load the imported mesh; list its skeleton's bone names (component/ref skeleton). **Assert set equality with the authority metadata** (all N native bones, no extras).
2. Assert parent indices equal the authority's.
3. Assert `mesh.skeleton` is the original native `Skeleton` object path (not a copy, not a new asset).
4. **Bind materials**: assign the owned material instances to slots by slot name; assert **no slot is left unbound** and none points at the engine default grid material.
5. Save the mesh and material assets; **reload** them from disk in the same script and re-check 1–4.
6. Report `owned_package_whitelist` (every package this stage created), `material_bindings`, native-material parent declarations (not shipped), protected-file hashes before/after.

**Trap:** during the session the native `Skeleton`'s `bone_tree`/`reference_skeleton` may *grow* if the import added bones. Never save the Skeleton asset. Verify, after the run, that the protected native skeleton `.uasset` **and** `.uexp` plus the `.uproject` are byte-identical to the pre-run hashes. A changed protected hash = the candidate is invalid even if everything else passed. [OBSERVED hazard; DECISION on the check]

**Trap:** FBX import prints bind-pose warnings in the log ("relative pose", matrix mismatch). A completed import with those warnings was a real defect, not noise. Treat any bind-pose warning as FAIL until the audit in §5 proves otherwise. (P3, docs/07)

## 5. Saved-bind audit (outside the editor) [OBSERVED procedure]

The editor's own in-memory view can hide a bad saved reference pose. The audit reads the **saved `.uasset` bytes** and compares them to the authority metadata.

Inputs: saved mesh `.uasset`, authority metadata (bone names, parents, **local and world** rest transforms), tolerances. Output: `saved-bind-audit.json`, `status: PASS|FAIL`.

Format facts for an uncooked UE 4.26 package (file tag `0x9E2A83C1`, legacy file version `-7`):

* There is no name-to-offset index for the mesh's `FReferenceSkeleton`. **Locate it by structure scan** over the export data:
  1. `int32 N` (bone count).
  2. `N ×` `FMeshBoneInfo`: `int32 name_index`, `int32 name_number`, `int32 parent_index`, then an export-name `FString` (`int32` length; negative → UTF-16 chars, positive → UTF-8 bytes incl. NUL).
  3. `int32 N` again, then `N ×` `FTransform` stored as **10 floats**: quaternion `xyzw`, translation `xyz`, scale `xyz`.
  4. `int32 N` again, then `N ×` name→index map entries `(int32 name_index, int32 name_number, int32 index)`.
* Require **exactly one** structurally valid match. Zero or several = FAIL (never pick the "closest").
* Validity filters while scanning: every parent `< own index` (or `-1` for the root), quaternions normalised within tolerance, name indices within the package name table.
* Resolve names through the package's name table; compare to authority names by index.
* Convert both representations to 4×4 matrices; compare **local** and **world** (accumulated) matrices with absolute `1e-4` and relative `1e-5` tolerance; also compare parents and names exactly.
* Hash the `.uasset` before and after; the audit is read-only.
* Observed values for a ~2,400-bone skeleton: max local delta ≈ `3e-7`, max world delta ≈ `1.5e-5` (float noise). Anything ≥ `1e-3` is a real mismatch.

Why both local and world: a world match with local mismatch means the importer re-solved the hierarchy; a local match with world mismatch means a wrong parent. Either breaks animation retargeting silently in game.

Run the audit **again** after the runtime-attachment restore (docs/11) because that stage re-saves the mesh. The final candidate's audit must postdate its last save. [OBSERVED: the final candidate's audit was re-run after runtime restoration]

## 6. Second, earlier audit in Blender (FBX bind audit) [DECISION]

Before spending editor time, a Blender script reads the exported FBX binary (`Pose`/`BindPose`, `LimbNode` `Lcl*`, `TransformLink`) and checks it equals the authority within tolerance (docs/07). That catches ~all bind errors in seconds instead of minutes. The editor audit then confirms the importer preserved it.

## 7. Failure → action

| Observation | Action |
|---|---|
| Bone set differs | Fix export (docs/05/07); never edit the skeleton to match |
| Local match, world mismatch (or reverse) | Parent/hierarchy error in FBX; fix export |
| Many `0`-scale or non-unit-quaternion transforms | Corrupt scan match; fix scan, not data |
| Protected hash changed | Discard candidate; re-clone project; find what saved the skeleton |
| Import timeout, no report, no log | Display problem (P13/P14), not the import |
| Report missing but exit 0 | The Python raised before writing; read `Import.log` |

Next: [docs/11 runtime attachments](11-runtime-attachments.md).
