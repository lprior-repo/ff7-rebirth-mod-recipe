# 15 — In-game verification: what to look at, how to automate it safely, and what *not* to claim

The only thing that proves a mod works is the game rendering and running it. Everything before this (docs/10–14) lowers the odds of a wasted test; none of it is a runtime pass. [OBSERVED: two candidates passed every offline gate and still rendered wrong / crashed.]

## 1. Preconditions

* An installed release with a receipt (docs/14), a fresh inventory, a backup, and **the human's approval to launch** (launching takes the GPU and display; some host services may need to stop, see §6).
* The release's inspection report is `PASS_…_CONTAINER_STRUCTURE_NOT_RUNTIME` for this exact hash.
* A written **observation plan** (the gates in §3) so you know what you are looking for before you look. Write expected-vs-observed, not just screenshots.
* The human, not the agent, starts the game from Steam unless they explicitly delegate it. [DECISION: launch options and Proton configuration belong to the user and stay unchanged.]

## 2. Guarded input automation (Linux/Wayland; spec, not code) [OBSERVED]

If the human delegates in-game inputs, drive them through one **guard** that enforces:

1. **Exactly one** window of the game's class (`steam_app_<appid>`) whose pid belongs to a running game process; otherwise send **no input**.
2. Focus that window via the compositor's dispatch API, then **verify** active-window pid *and* address equal the target **before every single input**; if focus moved, stop immediately.
3. Keys: `xdotool key --delay 150 --clearmodifiers <keysym>`; key names must match a strict regex (letters/digits/`_`/`+`); holds are **> 0 and ≤ 5 s**, with the key-up in a `finally`.
4. Mouse buttons: press, wait (poll frame time, do not busy-sleep blindly; ~0.2 s hold for the costume-selector Back, see P16), release — also with guaranteed release.
5. After input, **settle** (0.75 s default, ≤ 10 s), then capture **the game window only** (`grim -g "x,y WxH"` or equivalent) into `<label>-<runid>/window.png` plus an `observation.json` recording each operation, exit codes, window pid/address, timestamps. Label regex `[A-Za-z0-9_-]{1,80}`. New directory per run; never overwrite.
6. `observation.json` status is **`OBSERVED_INPUT_NOT_AN_AUTOMATIC_GAMEPLAY_PASS`** — the tool records, a human or explicit rubric judges.

On other desktops (X11/KDE/GNOME), substitute the equivalent active-window query and capture; the *invariants* (unique window, per-input focus check, bounded holds, immutable output) are the point. [GENERAL]

Hazards found: pressing Return repeatedly in a field scene did nothing visible and was followed by a crash (P17); holding a movement key for a long time then idling was followed by a crash ~4.5 min later. Neither crash was attributed (see §5). Don't mash inputs.

## 3. Gate matrix — one row per claim; unobserved rows stay "NOT OBSERVED"

| Gate | How to observe | Pass looks like | Source-project status (tested build) |
|---|---|---|---|
| Container loads, game reaches world | Launch with the release installed | World renders; no crash within N minutes | Observed (one candidate) |
| Appearance (identity/silhouette) | Full-body + close views, same camera as the approved render | Matches approved reference within the human's tolerance | Owner-judged; one earlier deployment **rejected** after viewing; later one rendered the approved model, not accepted as final |
| Materials | Under at least two lighting conditions; texture detail; no grid/gray/checker | Lit, textured, correct colours | Observed lit & coloured **after** native material cutover; earlier custom shaders gray/checkered |
| Deformation | Idle, walk, run, turn, arms raised, crouch/sit if reachable | No stretching, spikes, popping | Only idle/walk/run partially observed; no systematic pass |
| Grounding | Feet vs floor for heels/flat; shadow | No sinking/floating | **NOT OBSERVED** |
| Locomotion | w/a/s/d, sprint | Animation plays, root moves | Observed (walk) |
| Weapon draw / combat | Enter combat, draw, attack, abilities | Weapon attaches, no clipping catastrophe | **NOT OBSERVED** (weapon visible attached while exploring only) |
| Secondary physics (hair, chest, skirt) | Move, stop, turn | Plausible secondary motion | Hair moved; chest/skirt **NOT OBSERVED** |
| Cutscene / dialogue | Play a cutscene featuring the outfit | No face/hand/prop mismatch | **NOT OBSERVED** |
| Face / expressions | Dialogue close-ups | Lip sync/expressions plausible | **NOT OBSERVED** |
| Stability | ≥ 15 min mixed play; menu open/close; transitions | No crash | **Not met**: two crashes in short smokes (unattributed) |
| Rollback | Roll back by receipt, relaunch | Stock look, no leftovers | Receipt rollback performed and verified by file state; in-game relaunch after rollback **not recorded** |

Rules for the table: the status column comes from *recorded observations*, never from "it should". One screenshot proves "this was visible once", not "works".

## 4. Cheap judgement before expensive testing [DECISION]

The most expensive failure (hours of pipeline work on an appearance the human then rejected) was avoidable. Order work so the human sees:

1. **Blender renders** from fixed camera angles that match the reference images (docs/04) — approve *art* here.
2. **A thin vertical slice** (docs/17 S1): the simplest mesh through install and in-game *before* detailed modelling, to find pipeline defects while changes are cheap.
3. Only then in-game appearance, and compare to the *approved render*, not to memory.

## 5. Crash triage under Proton [OBSERVED]

* Collect: core dump timestamps, signal (`SIGSEGV`/`SEGV_MAPERR`), the **time since launch** and **time since last input**, the candidate/receipt id, any crash reporter output the game wrote. Store with the observation.
* Stacks from Wine crashes often show only `ntdll` frames (`save_context`, `segv_handler`) — **not attributable** to the mod. `gdb` on such cores printed BFD "segment extending past end of file" warnings and recovered no registers. Don't keep trying; record "unattributed".
* Discriminating experiments, in order of cost: (1) same session **with mod removed** repeating the same inputs (does the stock build crash too? the costume-selector Escape crash reproduced with all custom packages removed — it is *not* the mod); (2) same candidate, no input at all for the same time; (3) bisect the release by removing a package class (animations → materials → attachments) *only if* a crash reproduces. [INFERENCE: cause of the two unattributed crashes remains unknown; do not write it up as a mod bug or a mod innocence.]
* A crash is **not** proof of a mod defect, and a crash-free run is not proof of absence. Record frequencies, not stories.

## 6. Host housekeeping [OBSERVED]

* The test machine ran local-LLM GPU services. Stop **only** the one the human approved, only for the test, restore it after you confirm the game process exited, and **check its state again after any later launch** (a past restoration does not cover a later launch). Leave all other services alone.
* If the game window never appears, check the display socket (P13): a missing X11 socket under Xwayland broke Steam launches here; the fix was owner re-login. Don't restart the compositor or proxy the socket yourself.
* Only signal (SIGTERM) test processes you started or that are stalled test processes of this game; never `killall`.

## 7. What to write down after every test

`observation.json` (template in `templates/observation.json`) plus a ledger row (`templates/candidate-ledger.md`): candidate id, receipt path, what was launched, inputs sent, screenshots, **each gate row result with evidence path**, unexpected behaviour, host state changes (services stopped/restored), whether the game closed cleanly, and *explicitly what was not tested*.

Next: [docs/16 pitfalls catalog](16-pitfalls-catalog.md).
