# Rules for an AI agent using this recipe

1. **Read docs/01 first.** Exit code 0, a produced file, "import succeeded", an installed pak, or a control mod working is *never* a pass for the thing you built. A gate passes only when its checklist item has evidence that you inspected.
2. **Evidence tags.** Mark claims `[OBSERVED]`, `[DECISION]`, `[INFERENCE]`, `[GENERAL]`. Never write an unobserved claim as fact in reports.
3. **Immutable outputs.** Every stage writes to a fresh, unused output directory and refuses to overwrite. Never "fix" by reusing a path. Record input hashes with each output.
4. **Never write into the game directory except through the install procedure** in docs/14, with the game closed, a fresh inventory, a baseline backup and an ownership receipt. Never use the game's own Proton prefix for the editor.
5. **Do not ship what you only reference.** Native shared skeleton, native shader parents, native physics/driver assets referenced by your mesh must be *declared for the editor* and *excluded from the container* (docs/09, 11, 13).
6. **Separate art approval from technical gates.** An owner saying "looks good" approves appearance; it does not pass bind, cook, container, runtime, physics, animation or rollback gates.
7. **Run the thin vertical slice before heavy art work** (docs/17, stage S1). Prove the pipeline on a trivial change first.
8. **Blender skills / tooling:** if your environment provides Blender specialty skills, load the routing skill and every applicable specialty before Blender work; scene-specific invariants (native units, rest pose, UV/weight contracts) override generic tutorials.
9. **Do not rerun passed stages to re-verify them.** Reuse byte-identical outputs when the contract says reusable (docs/13 "reuse").
10. **Stop and report** on: version-gate mismatch, any unexplained hash change, any gate you cannot evidence, game still running, or anything that would mutate user saves/config/unrelated mods.
11. **Privacy:** this repo and anything you add must contain no extracted game data, mod files, keys, absolute home paths, account names, hashes of private assets or install receipts. Use placeholders (`<project-root>`, `<candidate-id>`).
12. **Pitfall hygiene:** when you meet a new failure, add an entry using `templates/pitfall-entry.md` with scope and tag, even if unresolved.
