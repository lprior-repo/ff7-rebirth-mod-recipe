# 10 — Delegation brief (sub-agents start blank)

A sub-agent's "done" is a **claim** until you check its report ([docs/01](../docs/01-evidence-discipline.md)). Delegate genuine, self-contained slices; keep top-level planning and integration yourself.

Fill every section:

```
# Goal / why
<one paragraph: what stage, what outcome, how it fits the ledger>

# Target
Inputs (path + sha256): ...
Fresh output directory (must not exist): ...
Non-goals: ...

# Required reading / skills
Docs: <recipe docs for this stage>
Blender work? Load the Blender skills the environment provides FIRST (also in this delegated task) and apply their procedures.
Project invariants: native-cm rig, frozen contract (bones/UV/vertex-group/slot names), ...

# Change / steps
1. ...

# Forbidden
No install/rollback; no launching the game; no stopping services; no deleting or overwriting existing files; no network downloads; do not rerun passed gates; do not edit the approved scene or ledger.

# Acceptance (observable)
- Report JSON at <path> with status in {PASS,FAIL,BLOCKED,NEEDS_ART_REVIEW}, input hashes, output list with hashes, failures list.
- Measured values for: <tolerances from docs>.
- Claims tagged [OBSERVED]/[INFERENCE]; unobserved aspects listed as NOT OBSERVED.

# Report back
Short: report path, status, deviations, anything surprising, anything skipped and why.
```

After it returns:
- [ ] Report file exists and has the expected status; input hashes equal pins.
- [ ] Output list == expected list.
- [ ] Read measured values; spot-check by re-running the cheap check yourself when the claim matters.
- [ ] Update the ledger from the report only.
