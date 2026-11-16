# 09 — In-game test session

## Before launch
- [ ] Human approved launching (the human starts the game unless delegated explicitly). Launch options and Proton settings untouched.
- [ ] Receipt + candidate + release hash written at the top of the observation file.
- [ ] Observation plan written (which gate rows, which scenes, which camera/pose set) with expected looks from the approved renders.
- [ ] Display socket present (P13); window class unique; input guard in place if inputs are delegated.
- [ ] If GPU headroom requires stopping a service: only the human-approved one, stop time recorded, restore planned.

## During
- [ ] Input operations bounded and logged: key regex, holds ≤ 5 s with guaranteed release, ~0.2 s hold for mouse Back in the third-party selector (P16), no mashing (P17).
- [ ] Per-input focus check on target pid + window address.
- [ ] Screenshot of **game window only** after each settle; labelled; immutable output dir.
- [ ] Observed rows only: appearance, materials, deformation (idle/walk/run/turn/arms raised), grounding, locomotion, weapon/combat, secondary physics, cutscene, face, stability, rollback.
- [ ] Same camera/pose as the approved render for the appearance comparison.
- [ ] Unexpected behaviour recorded as seen (no causal story).

## Crash handling
- [ ] Crash timing relative to launch and last input; signal; crash-reporter output; core-dump timestamps stored.
- [ ] Stack attribution attempted once; `ntdll`-only ⇒ "unattributed".
- [ ] Discriminating experiment proposed in order of cost: mod removed + same inputs; same candidate no input; selector bypass.

## After
- [ ] Game process confirmed exited; stopped services restored and their state re-read; state of any other launch noted.
- [ ] `observation.json` + ledger row: each gate row with evidence path or `NOT OBSERVED`.
- [ ] `gameplay_verified` left false unless the human decides otherwise after reviewing the matrix.
- [ ] Defect routed to the **cheapest** stage that can fix it (art → S2, packaging → repackage, materials → plan/instance, runtime links → attachments).
