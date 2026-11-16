# 05 — Game-source prep (Blender, before export)

Input: an **approved**, saved `.blend`. Output: a prepared, single-purpose `.blend` + `game-source.json` manifest, produced into a fresh directory. The input file's hash must be identical before and after.

## Algorithm (pseudocode)

```
open saved scene (hash pinned)
require exactly 1 armature
require armature bone-name set == authoritative metadata bone set
require each bone's parent == authoritative parent
require every visible mesh is bound to that armature (Armature modifier or parent)
if scene objects sit under a presentation-only parent (pure Z translation):
    re-express objects/rig in native cm coordinates; record height offset as RUNTIME requirement
clear actions/NLA mute; set pose to identity; set rig to REST
for each mesh:
    apply surface modifiers only (keep Armature); require show_viewport == show_render else error
    compact material slots to used slots only; error on an empty used slot
    per vertex: count groups with weight > 0 that map to a bone in the authoritative set
        unweighted vertex -> error; non-finite coordinate -> error
    report vertices with > 8 influences and total weight mass that would be lost
flag any Armature modifier using dual-quaternion (engine skins linearly)
delete everything not kept (lights, cameras, helpers)
set unit scale 0.01
save to output dir; verify source hash unchanged; write manifest incl. height_policy
```

## Rules behind it

* **Exactly one rig, exact bone set.** Extra helper bones or renamed bones fail the editor audit later. [OBSERVED design]
* **≤ 8 influences per vertex** (engine limit on tested build). More than 8 silently truncates; report the count and mass lost, and fix in Blender (limit total, normalize) rather than letting the importer decide. Observed model: 24 vertices with 5 influences passed a generic 4-influence check but are fine for 8. [OBSERVED]
* **Linear blend skinning only.** Dual-quaternion in Blender will not match the game. [OBSERVED design]
* **Slots by name.** Compaction keeps order; editor binds by slot name (docs/09).
* **Presentation offsets** are not part of the game asset (P22).
* **Source stays untouched**; the prepared copy is the thing exported.

## Manifest fields (suggested)

`status` (e.g. `PREPARED_NOT_COOKED`), source path+sha256, prepared blend path+sha256, metadata path+sha256, mesh count, vertex/triangle counts, per-mesh influence histogram, material slot list, height_policy, bone count, warnings (none hidden).
