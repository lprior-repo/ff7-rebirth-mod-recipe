# 18 — Continuing an existing mod (resume protocol)

For an agent dropped into a half-finished mod project. The failure mode here is the opposite of docs/17: not "declaring success early" but **re-doing passed work, trusting stale notes, or acting on a game state that has since changed.**

## 1. The first 30 minutes — read, don't run [DECISION]

1. **Entry docs, in this order:** the repo's agent rules → README (the selected workflow + exact safe commands) → the status file → the *specific* current report the status file names. Old work-logs are evidence, not instructions.
2. **Build a one-page state card** (put it in your scratch notes; template: `templates/candidate-ledger.md`):
   * Which candidate is *current*, which is *installed*, which are *historical/rejected*?
   * Which gates are PASS (with report path + input hash), which are NOT_RUN, which are FAIL?
   * What did the human approve, in what words, on what date, bound to which file hash — and what did they explicitly **not** approve?
   * What is the *last observed* game state (installed? game closed? services stopped?). Treat it as **stale** until refreshed (§3).
3. **Resolve contradictions explicitly.** Docs drift: a README may say "never do X" while a later status entry says the owner reversed it. The latest *dated* owner decision wins; record the conflict and the resolution in your state card. If two sources conflict and neither is dated, ask the human — this is exactly a "materially different tradeoff" question.
4. **List the do-not-repeat items** the project already states (e.g. "do not rebuild to repeat passed packaging diagnostics", "never duplicate/reimport cooked sequences", "do not rerun passed stages"). Those lines exist because someone paid for that lesson.

## 2. Classify the incoming task

| Task type | Cheapest correct entry | Do **not** |
|---|---|---|
| Appearance change (art) | docs/17 S2: new immutable scene from the *approved* parent; cheap renders; human approval; then the pipeline | tweak the deployed build; run the heavy pipeline before approval |
| Material/texture-only change | docs/08–09 plan translate → rebuild the **materials only**, reusing accepted geometry and movements (explicit "verified unchanged reuse" proof) | rebake geometry to debug a shader |
| Packaging/bulk-data defect | docs/13 **repackage** from verified cooked bytes | re-import or re-cook |
| Runtime attachment gap | docs/11 on the existing candidate lineage | assume attachments from the donor are present because the mesh looks right |
| Animation change | docs/12 only if native motion is insufficient | hand-duplicate/reimport cooked sequences (P6) |
| Install/rollback | docs/14 with a **fresh** inventory | trust an old inventory/receipt |
| In-game test | docs/15 with the human's launch approval | claim untested gates |
| New tool version / game patch | version gate first (docs/14 §2), then re-prove stage by stage | assume old pins still hold |

## 3. Refresh live state before any mutation [OBSERVED rule]

* Fresh read-only **inventory** (build id, exe hash, installed mod files, running processes, launch options, prefix). Compare with the ledger; any difference is news, not noise.
* Identify **the exact current receipt** (not "the latest in the folder") and its status. Roll back/advance by that receipt only.
* Confirm the game is closed (process list), not "was closed last time".
* Confirm host-service state if a previous test stopped one (P25): a past restoration does not establish state after a later launch.
* Check disk headroom — cooks and bakes are large and immutable-output rules mean they pile up. Don't delete evidence to make space; ask.

## 4. Candidate lineage and reuse [DECISION]

* Each candidate directory is immutable; a **new** candidate derives from a named parent candidate with a recorded relation: `geometry-from`, `movements-from`, `materials-from`.
* Reuse rules: copy bytes only after proving the file hash equals the parent's recorded hash; write a **reuse proof** (`parent`, `path`, `sha256 before == after`) into the new candidate's report. A rebuilt-but-identical thing is *not* proof of reuse; compare bytes.
* A failed or rejected candidate stays on disk as evidence and is marked in the ledger. It never gets "fixed in place".
* Never rename or move parents; reports embed paths and hashes.

## 5. Updating the ledger [OBSERVED discipline]

* Update status files **only from actual reports** — paste report path, status, hash; no paraphrase upgrades ("passed" → "works").
* Every new claim gets a tag (docs/01). Every unobserved gate stays literally `NOT OBSERVED`.
* When a human decision reverses an earlier one, write the new entry with date and *what it supersedes*; don't delete the old one.
* At the end of a session leave: current candidate, installed state, next gate, open defects, host state changes (services stopped/restored), and the single recommended next action.

## 6. Handling inherited uncertainty

| Inherited claim | What to do |
|---|---|
| "Gate passed" with no report path | treat as NOT_RUN until you find the report; if not found, re-run **that gate only** into a fresh output |
| Hash in notes ≠ file on disk | stop; the artefact drifted (see the converter-binary drift case: a rebuilt tool no longer matched the pinned hash and had to be rebuilt from source and re-verified) |
| "Looks right in game" (one screenshot) | record as a single observation; the gate matrix (docs/15 §3) still needs the other rows |
| "Crash was caused by the mod" | demand a discriminating experiment (docs/15 §5) before accepting or fixing |
| A script name implies purpose (a legacy-prefixed script reused for a different route) | read what it does; names may be legacy. Don't delete "legacy-named" shared helpers |
| Approval statement ambiguous | ask the human for a one-line, dated, hash-bound approval |

## 7. Delegating inside a resumed project

* Give the sub-agent: the state card, the exact stage, the do-not-repeat list, the output directory name (fresh), the evidence format, and a **forbidden actions** list (install, launch, stop services, delete, overwrite).
* If the project mandates loading Blender skills for Blender work, repeat that in the sub-agent's task; sub-agents start blank.
* Integrate their reports yourself; a sub-agent's "done" is *claimed*, not verified (docs/01).

## 8. When to stop and report

Stop and tell the human, with evidence, if: the game build id changed; a pinned tool hash changed; the ledger and disk disagree about what's installed; a gate that was recorded PASS fails on re-verification; or the next step needs launch/stop-service/install permission you do not have.
