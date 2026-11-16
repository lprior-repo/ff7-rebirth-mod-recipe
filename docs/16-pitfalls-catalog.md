# 16 — Pitfalls catalogue (symptom → cause → guard)

Every entry came from the source project unless tagged otherwise. **Scope** is: Steam build `23447986`, Rebirth V5 UE 4.26.2 editor, Blender 5.2.3, Proton, Linux/Hyprland. A different build may behave differently. Use `templates/pitfall-entry.md` to add entries; do not add one without a symptom you actually saw. Confidence: **C** = cause established by a test, **L** = likely/partial, **U** = unattributed (symptom and mitigation only).

Index by phase: Extraction P2 P4 · Blender/FBX P3 P22 P23 P26 · Materials P8 P9 P10 P11 P12 · Editor P5 P6 P14 P15 · Cook/pack P7 P8 P21 · Install P24 · Game P13 P16 P17 P18 P19 P20 P25 · Process P1 P19 P20.

---

## Process / evidence

### P1 — Exit code 0 is not success  (C)
* **Symptom:** a stage "finished" and the next stage later failed or produced wrong output (cook with a missing package; import with unbound materials; a commandlet that exits 0 after a script exception).
* **Cause:** Unreal commandlets and wrapper scripts often swallow errors; exit codes reflect process termination, not intent.
* **Guard:** every stage writes a report with `status`, input hashes, output list + hashes, `pending`/`failures`. The next stage reads and **re-verifies** it. Expected-vs-actual **set comparisons** for packages (docs/13). A timeout with no report and no log = environment problem first (P13/P14).

### P19 — Late art rejection  (C)
* **Symptom:** after weeks of pipeline work the human rejected the in-game appearance of a deployed model; an earlier model had been rejected the same way.
* **Cause:** technical gates were run on art that had never been approved on cheap, consistent renders; "approved" was inferred, then reversed.
* **Guard:** the human approves **fixed-angle renders** against the main reference *before* any editor/cook/install (docs/04, docs/17 S2). Record the approval as a dated, hash-bound decision (`approval = approve_current_for_game_test` ≠ gameplay acceptance). When the human supersedes an earlier rejection, record the new decision and keep the old one as history; don't re-ask settled questions, and don't silently restore a rejected build. Approval of art ≠ approval of physics/combat/stability.

### P20 — A control mod proves the route, nothing more  (C)
* **Symptom:** installing an unchanged author release "worked" and was mistaken for evidence the custom pipeline worked.
* **Cause:** the control exercises mount/load only; it uses the author's rig, materials, physics.
* **Guard:** label controls `UNCHANGED_AUTHOR_RELEASE`; never cite them as proof for your own assets (docs/14 §6).

---

## Extraction and data

### P2 — Extractor dropped 8-influence skinning and LODs  (C)
* **Symptom:** stock round trip altered deformation; only LOD0 existed; ≤ 4 influences per vertex.
* **Cause:** the quick PSK/umodel export format limits (docs/03).
* **Guard:** use the CUE4Parse route; **assert** influence count ≤ 8, non-zero weights sum to 1, LOD count equals native, against a second source (editor re-import).

### P4 — Container converter failed on a game-specific header word  (C)
* **Symptom:** `retoc` and `ZenTools-UE4` failed (ContainerHeader parse / `SerializeNum` assert).
* **Cause:** Rebirth's container header has one extra 32-bit word the stock parser doesn't expect.
* **Guard:** a converter fork at a pinned commit with a patch that accepts the extra-word count; verify the converted stock assets re-open in the editor. Record the commit, not just the name.

### P22 — Presentation offset is not game data  (C)
* **Symptom:** the authoring scene carried a ~+8 cm presentation offset (render/stage convenience) that is not part of the game asset.
* **Cause:** presentation placement mixed into the character's transform. [INFERENCE: the source project recorded the offset and excluded it; it did not observe it breaking an install.]
* **Guard:** keep presentation transforms on the scene root/empty only; assert the game-source root transform is identity and the bind pose matches authority (docs/05, docs/07).

### P26 — Old-format mesh layout  (L)
* **Symptom (risk):** community mesh patcher exists because earlier tool outputs used an older mesh layout.
* **Check:** run the patcher in "list" mode on the cooked output; freshly editor-cooked meshes classify as **new format** (no patch). If it says a patch is needed, you are using a stale export path. This ruled out one specific cause only; it is not a runtime proof.

---

## Blender / FBX

### P3 — FBX bind-pose warnings are a real defect  (C)
* **Symptom:** UE import reported bind-pose matrix warnings and a "completed" import; bones were subtly mispositioned.
* **Cause:** an FBX exported with armature/mesh bind poses not equal to the native rest poses (units, axis, parent-chain deltas).
* **Guard:** treat the warning as FAIL until the **saved-bind audit** (local+world deltas vs authority) is float-noise; generate explicit bind poses in the FBX post-patch (docs/07, docs/10).

### P23 — Blender 5.x action slots / API drift  (C)
* **Symptom:** animation-export code written against older Blender's action API needed rework on Blender 5.x.
* **Cause:** actions use **slots** (introduced in 4.4); channels live in slot-scoped channel bags. [GENERAL for the exact failure mode; the source project handled it by requiring a slot per action]
* **Guard:** pin Blender; bind the slot explicitly; smoke-test the exporter on one clip in a throwaway scene on every Blender upgrade.

---

## Materials

### P8 — Texture bulk not in the IoStore response  (C for the defect, **not** the cause of gray)
* **Symptom:** the container lacked `.ubulk/.m.ubulk/.uptnl` chunks; cooked texture/mesh payloads were incomplete.
* **Fix:** include those suffixes (never `.uexp`); verify each bulk chunk's payload hash + size against the cook report (docs/13).
* **Lesson:** after fixing this, the game **still** rendered gray/checkered (P12). The first plausible cause wasn't *the* cause; keep a hypothesis ledger.

### P9 — Material class change at an existing path requires rebinding BOTH meshes  (C)
* **Symptom:** after moving from UMaterial to MaterialInstanceConstant at the same object path, meshes pointed at stale/unresolvable materials.
* **Cause:** old imports don't resolve as the new class.
* **Guard:** rebind the outfit mesh **and** its condition mesh (docs/11); inspect imports in the saved and packed bytes.

### P10 — The editor stage accepted the raw bake manifest  (C)
* **Symptom:** the wrong (untranslated) plan could be fed to the import.
* **Guard:** a separate schema id for the translated plan; the editor stage rejects anything else (docs/09).

### P11 — Constant colours must be sRGB-encoded  (C)
* **Symptom:** solid colours rendered darker than the render.
* **Cause:** generated 1×1 bytes bypass the bake's linear→sRGB conversion.
* **Guard:** encode RGB (not alpha, not scalars), set `srgb=true` only for `Color` (docs/09).

### P12 — Custom UMaterial rendered gray/checkered  (C that native-parent instances fix it; mechanism within the engine **U**)
* **Symptom:** the candidate loaded, moved, but all surfaces gray/checkered (default/"missing shader" look) even with complete texture data (pixels decoded to correct colours).
* **Guard:** ship `MaterialInstanceConstant`s with **native parents** and native-named parameters (docs/09); inspect the packed container for parent identity.

---

## Editor / toolchain

### P5 — Editor Python can't write protected fields  (C)
* **Symptom:** notify event structs, `AssetUserData`, effect mesh refs couldn't be set via `set_editor_property`.
* **Guard:** console `SET <object> <Prop> <text>` with property text copied from the native serialization; always read back (docs/11, docs/12).

### P6 — Cooked AnimSequence duplicate/reimport is broken  (C)
* **Symptom:** duplicating or reimporting cooked native animations in the editor produced broken assets.
* **Guard:** stage immutable native baselines from a separate package root; import fresh targets at the native path; never duplicate cooked sequences (docs/12).

### P14 — Host X11 loss makes the editor hang  (C)
* **Symptom:** editor under Proton hung with no output.
* **Guard:** run the editor on a **dedicated** Xwayland/Xvfb display (`:99`), never the user's desktop session; set a timeout; no report + no log ⇒ display (docs/02).

### P15 — .NET dev-cert side effect  (C)
* **Symptom:** a .NET tool run created an ASP.NET developer certificate in the user's account as a first-run side effect.
* **Guard:** `DOTNET_GENERATE_ASPNET_CERTIFICATE=false` and other first-run opt-outs in the process env of every .NET call (docs/02). Don't delete such artefacts yourself; report them to the user.

---

## Cook / pack

### P7 — Listing `.uexp` in the IoStore response duplicated package IDs  (C)
* **Guard:** list only `.uasset/.ubulk/.m.ubulk/.uptnl`; assert chunk package ids are unique and equal the expected set (docs/13).

### P21 — Shipping native declarations or the native skeleton  (C)
* **Symptom (risk):** reference-only declarations at native object paths, or the shared skeleton touched by editor import, end up in the container and **shadow real game objects**.
* **Guard:** forbidden list in the cook whitelist check and in independent container inspection; verify on-disk hashes of the native skeleton unchanged after the editor run (docs/06, docs/11, docs/13).

---

## Install / game build

### P24 — A game patch invalidates everything  (L)
* **Symptom (risk):** a build change → cooked containers incompatible.
* **Guard:** `game_build_id` in the release; install refuses on mismatch; re-extract/re-cook after any patch (docs/14 §2). [INFERENCE: no patch occurred during the source project; the behaviour is expected, not observed.]

### P25 — Test-time service stops not restored  (C risk)
* **Symptom (risk):** a GPU service stopped for a test stays stopped after the game exits; later launch stops again.
* **Guard:** stop only the approved service, restore after **confirmed** exit, re-check state after any later launch (docs/15 §6).

---

## In-game

### P13 — Host X11 socket loss: Steam game shows no window  (C that the socket was missing; root cause **U**)
* **Symptom:** a Steam launch produced no window; runtime container warned the X11 socket path didn't exist; the socket directory was empty while a compositor's Xwayland still ran.
* **Resolution:** owner re-login restored it. [DECISION] Don't restart the compositor/Xwayland or proxy the path from an agent session; ask the owner.

### P16 — Escape in a third-party costume selector crashes the game  (C: reproduced with stock-only selection)
* **Symptom:** pressing Escape inside the selector's preview crashed the game, with **all custom packages removed**. Native Escape outside the widget worked.
* **Guard:** use right-mouse Back with a short (~0.2 s frame-polled) press; the guarded input tool encodes this. Upstream users reported the same keyboard crash. Never attribute it to your mod.

### P17 — Unattributed SIGSEGV under Wine  (U)
* **Symptom:** two short smoke runs ended with SIGSEGV (`SEGV_MAPERR`), minutes after launch/last input; cores had only `ntdll` frames; `gdb` yielded no registers.
* **Guard:** record, don't theorise: add the crash to the ledger with launch→crash times and last inputs; run discriminating experiments (docs/15 §5). Not a basis for "fixed" or "mod is innocent".

### P18 — Missing runtime attachments on a replacement mesh  (C measured; crash-causality **U**)
* **Symptom:** native mesh carried condition, Bonamik, KineDriver, physics-asset, effect refs; the first candidate had none.
* **Guard:** restore (docs/11) and inspect saved/cooked; record that it's a measured difference, not a proven crash cause.

---

## How to use this catalogue [DECISION]

1. Before each stage, read the entries for that phase (index at top).
2. When something fails, look for the **symptom** — but also run the entry's *Guard* checks, since the same symptom had multiple causes (gray materials: P8, P9, P10, P11, P12 were all different).
3. New failure → write a `pitfall-entry.md` with symptom, evidence path (private), scope, what's **known vs not**, and the guard. Mark unattributed items **U**.
