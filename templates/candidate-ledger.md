# Candidate ledger template (keep the filled copy in the private work tree)

The ledger is the single live record of what exists, what passed, and what is installed. Update it **only from actual reports**. Keep rejected candidates listed.

## Header
| Field | Value |
|---|---|
| Mod / slot replaced | `<char>_<outfit>` |
| Game build id | |
| Editor / Blender / converter pins | |
| Current candidate | |
| Installed candidate + exact receipt | |
| Game running at last check? | |
| Host services stopped (and restored?) | |

## Human decisions (dated, verbatim, never deleted)
| Date | Decision (words) | Bound to (file + hash) | What it is NOT |
|---|---|---|---|

## Candidates
| Candidate | Parent / reuse relation | Art scene (hash) | Bind audit | Materials | Runtime links | Animation | Cook set | Container inspection | Install receipt | In-game | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `<id>` | `<parent: geometry/movements/materials-from>` | | | | | | | | | | CURRENT / INSTALLED / HISTORICAL / REJECTED |

Cell format: `PASS <report> <short-hash>` / `FAIL <report>` / `NOT_RUN` / `NOT OBSERVED`.

## Gate matrix (in-game) for the installed candidate
| Gate | Result | Evidence | Date |
|---|---|---|---|
| Container loads / world reached | | | |
| Appearance | | | |
| Materials | | | |
| Deformation | | | |
| Grounding | | | |
| Locomotion | | | |
| Weapon / combat | | | |
| Secondary physics | | | |
| Cutscene | | | |
| Face | | | |
| Stability (≥ 15 min mixed) | | | |
| Rollback | | | |

## Do-not-repeat list
- `<passed gate> — report <path> — do not rerun unless inputs change>`

## Open defects
| Defect | Cheapest stage that can fix it | Owner |
|---|---|---|

## Next action (one)
`<action> — needs from human: <approval/launch/account step or none>`
