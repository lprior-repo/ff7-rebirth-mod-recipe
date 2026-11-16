# 14 — Release manifest, install, rollback, and the version gate

Goal: put an **inspected** release into the game directory in a way that is (a) refuse-on-doubt, (b) recorded by an ownership receipt, and (c) **exactly reversible** without touching anything the user or other mods own.

The source project implemented this as a small installer CLI. You may reimplement it in any language; the behaviours below are the spec. [DECISION]

## 0. How a replacement outfit reaches the game [OBSERVED]

* The mod **overrides a native outfit's asset paths** (pattern `/Game/Character/Player/<char>_<outfit>/Model/<mesh>` plus its materials/textures/optional animations). The mod's replacement mesh did load and render in the game from this location [OBSERVED]; that containers in `~mods` take priority over base containers is standard UE patch-mount behaviour [GENERAL]. No script, DLL or loader plugin is involved for a pure asset replacement.
* Community loader/menu/costume frameworks exist for this game (a runtime mod loader, a mod menu, a costume selector). **They are separate, third-party and optional** for an override mod. They matter for you because (1) the user may already have them installed and they change in-game UI/behaviour (P16: the costume selector crashes the game on Escape), and (2) installing *an unchanged author release* is the cheapest way to prove the install route works on the current build (P20). Honour their licences; do not repackage or redistribute them.
* Player-visible side effects: which outfit slot is replaced (e.g. the Standard outfit of one character) and any save that has it equipped. Say this plainly in the release notes.

## 1. Release manifest (`release.json`) — validation rules [OBSERVED]

Fields: `schema_version`, `release_id`, `stage`, `package_kind`, `game_build_id`, `candidate`, `source_hash`, `requires_plugins[]`, `files[{source, destination, sha256, bytes}]`, `gameplay_verified`. Template: `templates/release.json`.

Validate **before** anything touches the game directory:

| Rule | Why |
|---|---|
| `stage` ∈ {`COOKED`, `UNCHANGED_AUTHOR_RELEASE`, `FORMAT_CONVERTED_AUTHOR_RELEASE`} | refuses to deploy authoring outputs (Blender/FBX/editor project) |
| every `source` is relative, canonical: no `..`, no backslash, no drive colon, not absolute; source is a regular file, **not a symlink** | path traversal / link escapes |
| every `destination` is either under `End/Mods/<name>/…` or exactly `End/Content/Paks/~mods/*.pak|*.utoc|*.ucas` | only approved directories |
| no two destinations equal **case-insensitively** | Wine/Windows case folding collides `Foo` and `foo` |
| no destination component is a symlink; destination does not already exist | no overwrite, ever |
| file sha256 equals the manifest | source tampering/corruption |
| each pak/utoc/ucas stem has a **complete companion set** (`.utoc` needs `.ucas`, and vice versa) | half a container is a crash |
| `requires_plugins[]` names are simple identifiers and each is **installed** | e.g. an author release that needs a loader |
| `game_build_id` equals the installed game's build id | version gate (§2) |

## 2. Version gate — a fresh inventory before every mutation [OBSERVED]

A read-only **inventory** is a private JSON recording: Steam app manifest (build id, hash), game executable path + sha256, compat-tool mapping, launch options, Proton prefix + runtime versions, all existing mod files with hashes (paks, plugin dirs, loader DLLs such as `dwmapi`/`dinput8`/`xinput1_3`/script-extender DLLs), GPU/memory, and running game processes. Keep it **local/private**.

Install refuses unless: inventory is fresh (generated in this session), executable hash still equals the inventory (the game did not patch itself), `release.game_build_id == inventory.build_id`, and **no process whose `comm` starts with the game binary name is running** (read `/proc/*/comm`; do not rely on Steam's status). A game patch silently invalidates cooked containers (engine/serialization changes) and may rename paths — re-cook, don't just bump the id. (P24)

## 3. Pre-install backup [OBSERVED]

Back up what the game and launcher keep outside the install: save directory, game config, existing mod files, Steam/compat settings. In the source project this was a hashed 16-file baseline (saves + config + loader files) written before the first install. A mod install does not normally write there; the backup exists so that **when a test goes wrong the human has a known-good restore** that is independent of your rollback code. Do it once per milestone, not per candidate; verify by hash.

## 4. Install algorithm [OBSERVED]

1. Require: game closed, fresh inventory (§2), validated manifest (§1), inspection report `PASS_…` from docs/13 for **this exact release hash**.
2. Create a **0700 transaction directory** and an **empty receipt path**; refuse if either exists. Copy every source file into the transaction dir; re-hash each; refuse if the source changed mid-copy.
3. Write the receipt (`status: STAGING`), then set `INSTALLING`.
4. For each file: create missing destination directories (record exactly which were created); **hard-link** the staged file to the destination with `os.link` (atomic; fails if the destination exists = no overwrite, no TOCTOU replace). Record `device`, `inode`, sha256, size per file.
5. Re-hash installed bytes against the manifest. Mismatch → `INSTALL_FAILED` and roll back what this receipt created.
6. Set receipt `INSTALLED`, `gameplay_verified: false` (it becomes true only by a human decision after docs/15; do not set it from a smoke).

Why hardlinks: the same inode lives in the transaction dir and the game dir, so rollback can prove "this is still the file I installed" (inode + hash) and a user edit/replace is detectable. [DECISION] Hardlinks require the same filesystem. [INFERENCE: the source project's game and transaction dirs shared a filesystem; if yours do not, copy and record the inode of the copy instead — untested.]

## 5. Rollback algorithm [OBSERVED]

Only by **receipt**; never "delete everything named like our files".

1. Refuse if receipt is already `ROLLED_BACK`; refuse (state a reason) if the game is running.
2. For each owned file: current `device/inode` and sha256 must equal the receipt. If **not**, abort the whole rollback with *"user-modified file preserved"* and leave everything as it is.
3. `rename` each file into a quarantine directory (atomic, same filesystem), then verify the quarantined bytes; refuse if a quarantine name collides. Concurrent change during rollback → stop and report the quarantine path.
4. `rmdir` only directories **this receipt created**, and only if empty.
5. Status `ROLLING_BACK` → `ROLLED_BACK` (or `ROLLBACK_FAILED`). Keep the receipt, the failed candidate and any crash dumps for diagnosis.

Rollback restores the *ownership set* only. It does not restore a **different** release; to switch candidates: roll back, inventory, install the new one. [DECISION] Prove rollback on a throwaway/control release before relying on it for a real one; the source project did exactly this order (control install + rollback, then the outfit).

## 6. Unchanged author-release controls [OBSERVED]

For third-party framework/outfit controls, a "stage" step unpacks the pinned author archive into a release tree **without altering it** (checks: archive members relative and canonical, no symlinks or special files, plugin directory name equals descriptor name, no case-fold collisions; stage = `UNCHANGED_AUTHOR_RELEASE`, `requires_plugins` from the descriptor) and then the same install/rollback applies. Pin the author release by archive sha256 in your own *private* config; never commit the archive.

A passing control proves only: the mount route works on this build/Proton setup. It does **not** prove your mesh, materials or physics (P20). Also record permission/licence limits stated by the author in the release notes (some forbid modification or reuse — do not feed such assets into your pipeline).

## 7. Efficient iteration loop [DECISION]

```
candidate N inspected PASS  ──►  inventory + game closed?  ──►  rollback N-1 receipt
        ──►  install N  ──►  observe (docs/15)  ──►  record result in the candidate ledger
```

Keep at most **one** candidate installed. Name containers with the candidate id so `ls ~mods` tells you what is live. Record the receipt path in the ledger *before* launching the game.

Next: [docs/15 in-game verification](15-in-game-verification.md).
