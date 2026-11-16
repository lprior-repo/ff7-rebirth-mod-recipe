# 19 — Sources: upstream tools and where to get them

Only public upstream locations are listed. Versions are what the source project used (`[OBSERVED]` working); newer ones are untested. **Verify each tool's licence before use and honour it.** Nothing here is hosted in this repo.

| Tool | Upstream | Tested | Used for | Licence note |
|---|---|---|---|---|
| Blender | https://www.blender.org/ (release archive: https://download.blender.org/release/Blender5.2/) | 5.2.3 LTS (5.2.2 earlier) | authoring, prep, bake, FBX export, headless scripts | GPL-3.0-or-later; verify the published archive checksum from the release page |
| CUE4Parse (+ Conversion) | https://github.com/FabianFG/CUE4Parse ; NuGet packages `CUE4Parse`, `CUE4Parse-Conversion` | 1.2.2.202610 | parse game containers, mesh/anim/material export | Apache-2.0 (the version used); re-check on upgrade |
| SixLabors.ImageSharp | https://www.nuget.org/packages/SixLabors.ImageSharp | 3.1.12 pinned | image decode inside the extractor | [OBSERVED] the project's notes: 3.1.12 has published advisories; 4.x needs a commercial licence, so it was not used. Re-check both facts and decide knowingly |
| .NET SDK | https://dotnet.microsoft.com/download/dotnet | 10.0.401 (project-local) | build/run extractor | MIT; set `DOTNET_GENERATE_ASPNET_CERTIFICATE=false` (P15) |
| retoc (IoStore↔legacy converter) | https://github.com/trumank/retoc | upstream 0.1.5 failed on this game; a commit-pinned fork + small patch worked | convert game IoStore packages to editor-readable legacy assets | check upstream licence; the patch only added a flag for the container header's extra package-count word (P4) |
| ZenTools-UE4 | https://github.com/WistfulHopes/ZenTools-UE4 | 1.0.3 **failed** here | alternative converter | recorded as a negative result only |
| FFVII Rebirth Mesh Patcher | https://github.com/nikolaybutnik/FFVII-Rebirth-Mesh-Patcher | 1.7.2 | classify meshes as old/new format (check-only use) | read its notes on format versions (P26) |
| UE4SS | https://github.com/UE4SS-RE/RE-UE4SS | 3.0.1 (experimental build) | dump reflection data to generate a usmap for the running game | MIT; used only for mapping generation, not shipped |
| Rebirth V5 editor project | community project release tagged `FF7RProjectV5` at https://github.com/albixh/FF7R2UProj/releases (editor engine source is a private, account-gated repository; access requires being a member of the relevant Epic Games GitHub organisation) | UE 4.26.2 | import, bind, cook, IoStore, UnrealPak | **licence not established by the project archive** — treat as private use, do not redistribute it |
| Proton | Valve; use the full Proton, not bare Wine (missing D3D deps) | Experimental | run the Windows editor in its own prefix | — |
| io_scene_psk_psa | https://extensions.blender.org/add-ons/io-scene-psk-psa/ | 9.1.3 | PSK/PSA import (only for umodel exports; not needed with a native extractor) | GPL-3.0-or-later extension |
| UE Viewer (umodel) | community Rebirth build (forum thread listed by the project) | — | quick previews | lossy for this work (≤4 influences, LOD0 only, P2) |
| Community mod loader / mod menu / costume selector | Nexus Mods → Final Fantasy VII Rebirth → Mods (search by role) | — | optional runtime frameworks for installing/selecting mods | third-party; read each author's rules; their menu has a known Escape crash that reproduces without your mod (P16) |

## Generic references (not project-specific) [GENERAL]

* Unreal Engine 4.26 docs: skeletal mesh import, material instances, cook commandlet, UnrealPak/IoStore command-line.
* Blender manual: FBX export options, action slots (4.4+), unit scale, headless scripting (`--background --python`).
* Steam on Linux: app manifest (`appmanifest_<appid>.acf`) holds the build id; compat-tool mapping in Steam's `config.vdf`; launch options per app.

## Not included here — and why

* Game extractions, mappings, keys, saves, models, textures, FBX/PSK, editor project, cooked outputs, installers, receipts, inventories, hashes. All local-only; keep it that way.
* Any third-party mod archive. Cite its public page; do not copy it.
* Any account credential or token. The editor-access step requires the human's own authorised account.
