# 03 — Native extraction (getting the game's own data)

You need the game's native skeleton, rest transforms, sockets, material slots, skinning, LODs and animation timing as **authoritative data**. Everything later is checked against it.

## Route that worked [OBSERVED]

A small C# extractor built on **CUE4Parse** (game profile `GAME_FinalFantasy7Rebirth`) with:

* the game root pointed at the Steam install (read-only);
* a **usmap** mappings file generated from the running game for the same build (needed because the game uses unversioned properties; without it, property parsing fails);
* Oodle initialised from a local library path (never the downloading overload);
* AES keys only from a user-supplied key file if archives are encrypted (not recorded anywhere).

Modes the extractor needed (design them as separate sub-commands): `inspect` (list/find assets by substring), `export` (mesh), `materials`, `animation-info`, `object-info`, `texture-info`.

## What to extract and record (the "authoritative metadata" JSON)

| Field | Why |
|---|---|
| `authoritativeSkeletonBones[]`: name, parentIndex, restTransform (translation, quaternion, scale) | bone set, parent-first order, rest pose for FBX patch and bind audit |
| Mesh bone list (subset/superset of skeleton) | stock mesh used a subset of the shared skeleton's bones |
| Sockets (name, bone, relative transform) | the weapon attaches by socket; record all of them |
| Material slots (names, order) | material binding is by slot name |
| Per-vertex influences (up to 8), all LODs | see below |
| UV channels count | record the stock count; keep it |
| Dependency package list | informs what to reference vs own; expect a long list |
| Animation: sequence paths, frame times, rate, root ("Trans") per-frame transforms, notifies, curves, flags | animation replacement needs exact timing/root |
| Physics asset, kine-driver, bonamik (secondary motion) and effect references on the mesh | see docs/11 |

Record counts per stock character in your private ledger (skeleton bones, mesh-influence bones, sockets, material slots, LOD0 vertices, UV channels, LOD count). Do not copy the numbers into this repo: they are extracted-game-derived values and are specific to one character and build. [GENERAL]

## Why not umodel / PSK [OBSERVED]

umodel's FF7 Rebirth export kept **≤4 influences per vertex and LOD0 only**. A stock round trip then fails preservation (changed deformation, missing LODs). Use it for quick eyeballing only. If you must use PSK/PSA in Blender, the Blender extension `io_scene_psk_psa` 9.1.3 imported them, but you inherit the same limits.

## Legacy-container conversion [OBSERVED]

Converting IoStore-format game assets to legacy `.uasset` for the editor failed with **retoc 0.1.5** (ContainerHeader parse errors) and **ZenTools-UE4 1.0.3** (SerializeNum assert). A retoc **fork at a specific commit plus a small patch** adding a `--container-header-extra-package-count-word` flag converted native assets correctly: Rebirth's container header has one extra 32-bit word the stock parser does not expect. [INFERENCE: the extra word is a package count; confirmed only by the fix working.]

## Pitfalls

* Extract from a **copy-free, read-only** view; never write into the game directory.
* Record extractor/mapping/game build in every metadata file; re-extract after a game update.
* Do not store keys; do not commit extracted data.
* The ImageSharp advisories (see docs/02) matter only if you decode untrusted images; texture inspection used it on game assets. Pin and note.
* Check, don't assume, that the extractor preserved influences and LOD count by comparing to a second source (e.g. editor import of the same mesh) before building on it.
