# 13 — Cook, package (IoStore), and independent container inspection

Inputs: a saved candidate project that passed the saved-bind audit (docs/10), runtime attachments (docs/11, if used), native-material instances (docs/09), optional animations (docs/12). Output: an immutable **release** — a set of `.utoc/.ucas/.pak` files plus a manifest — that has been inspected **from the container bytes**, not from the editor.

## 0. Principles [DECISION]

* **Everything is immutable and per-candidate.** One candidate id → one fresh directory tree. Never overwrite or "fix" a candidate; make a new one. Reuse = copy-with-hash-verification into a *new* candidate.
* **Whitelist, not glob.** The set of packages you ship is an explicit, sorted list assembled from the import report (owned mesh/material/texture packages) + runtime extras (condition mesh) + animation targets. Anything that would ship but is not on the list is a failure, and so is anything on the list that did not appear.
* **A stage's exit code is not its result.** Each stage writes a report (status string, input hashes, outputs with sha256 + size, `pending`/`failures` flags). The next stage reads the previous report and re-hashes the inputs. (docs/01, P1)
* **Statuses say what was *not* proven.** Use `..._PENDING_INDEPENDENT_INSPECTION`, `..._CONTAINER_STRUCTURE_NOT_RUNTIME`, `gameplay_verified: false` until a human has seen the game.

## 1. Candidate init

`init` creates the project skeleton for a candidate, copying in (by hash) the pinned editor project template and the source package. Record: candidate id, source model hash, metadata hash, native skeleton path, engine version hash, game build id. These are the **identity keys**; every later stage and every reuse compares them. [OBSERVED]

## 2. Cook

Editor commandlet under the matched editor + Proton (dedicated display, docs/02 P14):

```
<editor> <project> -run=Cook -TargetPlatform=WindowsNoEditor -CookSinglePackageNoRefs \
         -Map=<pkgA>+<pkgB>+... (sorted, '+'-joined) -abslog=<output>/Cook.log \
         -NullRHI -unattended -NoSplash -NoSound -NoP4 -stdout -FullStdOutLogOutput -UTF8Output
```

* `-CookSinglePackageNoRefs` cooks **only the named packages**, not their reference closure — essential, otherwise the cook drags in native game assets (skeleton, parent materials) that must not ship. `-Map` takes the whitelist `+`-joined and **sorted** (deterministic).
* The cook stage **refuses to start** if the whitelist intersects `native_references_not_shipped` (docs/11) or if the saved inspections are missing.
* Afterwards compute expected = `{Content/<pkg>.uasset}` for the whitelist vs actual = recursive listing of the cooked output. Missing **or** unexpected → FAIL. Write a cook report listing **every cooked file with path, sha256, bytes**. [OBSERVED]
* Exit code 0 with a missing package is common (P1). The set comparison is the gate.
* Cooked `.uasset/.uexp/.ubulk/.m.ubulk/.uptnl`: the `.ubulk`/`.m.ubulk` (bulk data: mesh vertices, textures) and `.uptnl` files must be accounted for. Large meshes/textures put bytes in bulk files; losing them yields a loadable but empty/gray asset (P8 — note: in the source project missing bulk was a real defect that was fixed, but it was **not** the cause of the gray-material symptom; do not stop investigating at the first plausible cause).

## 3. Package to IoStore

Rebirth loads IoStore containers (`.utoc` + `.ucas`) from `End/Content/Paks/~mods/`. [OBSERVED]

1. **Response file** (for `UnrealPak`/IoStore): one line per file, quoted `"<cooked path>" "../../../<relative game path>"`, UTF-8 **with BOM**, sorted. Include suffixes `.uasset`, `.ubulk`, `.m.ubulk`, `.uptnl` for each package on the whitelist. **Never list `.uexp`** — IoStore consumes it implicitly; listing it creates duplicate package ids and a broken container (P7).
2. **Commands file:** `-Output="<…>.utoc" -ContainerName=<stem> -ResponseFile="<resp>"`.
3. Run the engine's IoStore commandlet: `-run=IoStore -CreateGlobalContainer=<scratch>/global.utoc -CookedDirectory=<cooked> -Commands=<commands file> -TargetPlatform=WindowsNoEditor`. The scratch `global.utoc/.ucas` is a build artefact and **must not be shipped** (it would replace the game's global container).
4. Output must contain the `.utoc` **and** `.ucas`. Pick a container stem `"<ModName>_<candidateId>_P"` (`_P` is the usual UE patch-container suffix so it mounts after base containers [GENERAL]; the stem pattern itself was used in the source project [OBSERVED]).
5. **Provenance:** a tiny companion `.pak` (created with `UnrealPak.exe <pak> -Create=<resp>`) holding one JSON file mounted at `../../../End/Content/<ModName>/BuildInfo/<json>` records candidate id, source hash, cook-report hash and `gameplay_verified: false`. This makes an installed container identifiable from inside the game directory. The game does not need it to run.
6. **Release manifest** (`release.json`, template in `templates/release.json`): `stage: COOKED`, release id, `package_kind`, **game build id**, candidate id, source hash, `files[{source, destination, sha256}]` with destinations only under `End/Content/Paks/~mods/`, `gameplay_verified: false`.
7. **Package report:** `status: PENDING_INDEPENDENT_INSPECTION`, `bulk_files[]`, and an **`excluded` list** stating what was deliberately *not* packaged: native shared skeleton, import-only skeleton, scratch global container, native AssetRegistry.

## 4. Independent container inspection (the real gate)

Use a **different tool from the one that produced the bytes** (the project's patcher libraries for IoStore/zen/CityHash, or CUE4Parse). Inputs: the release manifest and the container files. Assert, all of:

**Release integrity**
* Every release file hash equals the manifest hash; exactly **one** `.utoc` (and its `.ucas`); no extra files.

**Container structure**
* The package-id set in the TOC chunks equals the **expected whitelist set** exactly, no duplicates.
* **Forbidden** (must be absent): native skeleton, import-only skeleton, native declarations (docs/11), native material-parent declarations (docs/09).
* Bulk chunks (chunk types 3, 4, 5: bulk data / optional bulk / memory-mapped bulk): every bulk chunk matches **exactly one** expected bulk file; its decompressed payload's sha256 and size equal the cook report; no extra bulk chunk.
* Every chunk's **SHA-1 equals the TOC meta hash** (catches corrupted compression).
* Each package's **public export name** and its **global import id** are correct (the ids the engine will use to resolve the package at runtime).

**Per-asset semantics**
* Mesh exports are `SkeletalMesh`; their imports reference the **native skeleton exactly once**; each material import occurs **exactly once**; **no default-grid material import** (`WorldGridMaterial`): its presence means a material slot failed to bind at cook time and the game will render a grid or gray.
* Runtime meshes (docs/11): native references imported once; exactly one KBD user-data export and one condition user-data export.
* Materials are `MaterialInstanceConstant` whose parent is a **native** parent; textures imported once; parameter values decode to the plan (docs/09). Reject packaged declarations/static overrides.
* Animation packages: `AnimSequence`s with native skeleton import once; timing/root/curve checks as in docs/12.

**Report:** `status: PASS_<KIND>_CONTAINER_STRUCTURE_NOT_RUNTIME`, a `limits` list ("does not prove loading, rendering, physics, combat"), and `runtime_verified: false`. [OBSERVED] Only this report allows install.

**Mesh-patcher sanity ("list" mode):** a freshly cooked mesh is already the current layout ("new format"); the old-format patcher must report "no patch needed". If it says a patch is needed you are using an old-layout export (P26).

## 5. Repackaging without recooking (reuse)

When only packaging changes (a container-stem rename, bulk-file inclusion, manifest field), do **not** recook:

1. Verify every cooked file hash from the source candidate against its cook report.
2. `init` a fresh candidate; assert **identity keys equal** (source hash, metadata, mesh asset path, skeleton path, engine hash, game build id). Mismatch → stop.
3. Copy the cooked tree (never overwrite); copy prior reports with their hashes; write a `repackage-provenance` naming the source candidate and what changed.
4. Package again, inspect again. A repackage is a **new release id**.

Same idea for "unchanged-movement" reuse of cooked animations (docs/12 §Reuse).

## 6. Efficiency notes [OBSERVED]

* [INFERENCE: timings were not logged precisely] Cook + IoStore of one mesh/materials/textures set takes minutes, and editor startup under Proton was the visible overhead. Batch all packages for a candidate in **one** cook invocation.
* The independent inspection is a seconds-long pure-Python/.NET job. Run it after every packaging, not only at the end.
* Keep the cook report as the single source of truth for hashes: the inspection compares *to it*, not to anything it re-derives.
* A packaging bug found only in the live game cost the most. Catch it here (duplicate package ids, missing bulk, default grid material, forbidden native asset).

Next: [docs/14 release, install, rollback](14-release-install-rollback.md).
