# 01 — Evidence discipline (read before acting)

Most wasted time in the source project came from believing a stage had passed. These rules are the cure.

## Tags

| Tag | Meaning |
|---|---|
| `[OBSERVED]` | Seen in a tool output, report, or screenshot in the source project, on the tested versions |
| `[DECISION]` | A design choice that worked; alternatives may too |
| `[INFERENCE]` | Plausible explanation, not proven. Always say what would prove it |
| `[GENERAL]` | Standard practice not specifically tested here |

## Non-pass list (things that are NOT proof)

* Process exit code 0 (editor commandlets can exit 0 after logging an error).
* A produced file exists.
* An FBX/asset import "completed" (bind-pose warnings were real defects).
* A cook completed (package set may differ; bulk data may be missing from the pack).
* A control mod (a known-good third-party mod) installs and works → proves the *install route*, not your outfit.
* Inventory / listing of installed files.
* A human saying "looks good" (approves *appearance*, not bind/runtime/physics).
* Decoding texture pixels from the container (proves data present; not UVs, shader compatibility, or in-game look).
* A single in-game screenshot (one pose, one scene).
* Absence of a crash in a short session.

## Pass requires

1. The report file for the stage exists, with `status` in the expected pass vocabulary, **and** you read the pending/failure flags.
2. Input hashes in the report equal the hashes you intended (pins).
3. The output list equals the expected list (missing *and* unexpected both fail).
4. For numeric checks: the measured value, the tolerance, and the margin.
5. For in-game checks: a per-gate observation (what you did, what was visible, what is still unobserved).

## Status vocabulary

`PASS`, `FAIL`, `NOT_RUN`, `BLOCKED`, `NEEDS_ART_REVIEW`. Never invent softer words ("mostly passed"). Record limits explicitly: "passes for N bones/clips/materials; X not covered".

## Immutable-output rule

Every stage gets a new output id. If it exists, error; never overwrite. A failed run's directory is kept as evidence. Repeating a stage means a new id with new report; reuse of earlier bytes happens only through an explicit "verified unchanged reuse" proof (docs/13).

## Ownership of decisions

* The human owns appearance approval and risky actions (installing into the game, stopping services, launching the game).
* The agent owns technical gates and must not downgrade them to please a deadline.
* Approval of art **supersedes** earlier rejection of a *different* scene; it never converts a technical gate to PASS.

## Reporting format (per stage)

```
stage: <name>   status: PASS|FAIL|...   inputs: <ids + hashes>
checked: <list>   measured: <numbers + tolerance>
not covered: <list>   next: <one step>
```
Use `templates/stage-report.json` for the machine-readable form.
