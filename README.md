# FF7 Rebirth character-mod recipe

A documentation-only playbook for building a **custom character-model mod for Final Fantasy VII Rebirth (PC, Steam app 2909400) on Linux**, written so an AI coding agent (or a human) can follow it start to finish without re-discovering the pitfalls.

It was distilled from one long, real attempt: replacing a main-character outfit mesh, its materials and nine animations, cooking them with a matched Unreal Engine 4.26 editor under Proton, installing them as IoStore containers, and trying to verify them in the live game.

## What this repo is — and is not

| It is | It is not |
|---|---|
| Procedures, gate definitions, spec tables, pseudocode, checklists, templates | A library, CLI or scripts. There is **no executable code** here, on purpose |
| A catalogue of observed failures with their scope (version/build/what was seen) | A promise that any step works on other game builds or tool versions |
| Independent of any one character, outfit, mod or owner | A copy of any game asset, mod asset, model, texture, hash, receipt or private path |

Every non-obvious claim carries an evidence tag (see [docs/01](docs/01-evidence-discipline.md)). Anything tagged `[INFERENCE]` or `[GENERAL]` was **not** observed in the source project; verify it before relying on it.

## Tested scope (everything is "as tested")

* Game: Steam app 2909400, Steam build id `23447986`, Windows build run through Proton on Linux.
* Editor: community "Rebirth V5" Unreal Engine 4.26.2 editor build (engine version string `4.26.2-0+++UE4+Release-4.26`), run under Proton in its own prefix.
* Blender 5.2.3 LTS (bundled Python 3.13); host Python 3.14; .NET SDK 10.0.401; CUE4Parse 1.2.2.
* A newer game patch, engine build, or Blender major version may break any of the above. Re-prove the version gate ([docs/14](docs/14-release-install-rollback.md)) first.

## How to read it

1. [docs/00-overview.md](docs/00-overview.md) — the pipeline on one page and where the time actually went.
2. [docs/01-evidence-discipline.md](docs/01-evidence-discipline.md) — rules that stop an agent from declaring success early. **Read before acting.**
3. [docs/17-new-mod-playbook.md](docs/17-new-mod-playbook.md) — the ordered plan for a brand-new mod, with a thin-vertical-slice-first strategy.
4. [docs/18-continuing-an-existing-mod.md](docs/18-continuing-an-existing-mod.md) — what differs when you resume a half-finished mod rather than start new.
5. [docs/16-pitfalls-catalog.md](docs/16-pitfalls-catalog.md) — every observed failure, symptom → cause → fix.
6. The topic docs 02–15 as each stage comes up; [checklists/](checklists/) for gate sign-off; [templates/](templates/) for evidence records.

## Layout

```
AGENTS.md              operating rules for an AI agent using this recipe
docs/00..19            the recipe
checklists/            gate checklists (copy into a work log, tick with evidence)
templates/             JSON/Markdown record shapes (not code)
```

## Legal / safety posture

Modding a commercial game: keep game assets, extracted data, third-party mod files, keys and saves **local**. This repo intentionally contains none. Do not use it to bypass DRM or redistribute game data. Third-party tools named here are cited with upstream locations in [docs/19](docs/19-sources.md); honour their licences.
