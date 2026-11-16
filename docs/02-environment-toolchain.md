# 02 — Environment and toolchain (as tested)

All versions below are `[OBSERVED]` working on the tested workstation (Arch Linux, x86-64, Hyprland/Wayland with Xwayland, NVIDIA GPUs, Steam + Proton). Pin and record yours; do not float.

## Components

| Component | Tested version | Role | Notes |
|---|---|---|---|
| Blender | 5.2.3 LTS (earlier 5.2.2 also used), bundled Python 3.13 | authoring, prep, bake, FBX export | Scripts importing `bpy` must run under Blender's own Python, never the project venv |
| Host Python | CPython 3.14.8 in a project venv | orchestration, hashing, JSON | no `bpy` here |
| .NET SDK | 10.0.401, project-local | NativeExtractor (CUE4Parse) | set `DOTNET_GENERATE_ASPNET_CERTIFICATE=false` (see pitfall P15) |
| CUE4Parse | 1.2.2.202610 (+ Conversion) | parse game containers, skeletal mesh, anims, materials | ImageSharp pinned exactly 3.1.12; it has published advisories; 4.x requires a commercial licence → not used |
| Rebirth V5 UE editor | UE 4.26.2 (community fork tagged `FF7Rebirthv5`; project `FF7R2UProj-V5`) | import, bind, cook, IoStore, UnrealPak | author-declared compatibility with game v1.005. Source is a **private** GitHub repo requiring membership in the Epic Games GitHub org (link GitHub↔Epic account, accept invite) |
| Proton | Experimental (a full Proton; bare Wine failed on missing D3D dependencies) | run the Windows editor | **dedicated prefix**, never the game's |
| retoc | 0.1.5 failed; fork of upstream at a specific commit + small patch worked | convert IoStore↔legacy assets | patch adds a flag `--container-header-extra-package-count-word`; Rebirth's container header has one extra word |
| ZenTools-UE4 | 1.0.3 failed | alternative converter | ContainerHeader parse errors / SerializeNum assert |
| umodel (UE Viewer) | FF7 Rebirth v5 build (forum topic 9160) | quick preview/export | **lossy for this purpose**: ≤4 influences/vertex and LOD0 only → fails stock-preservation |
| UE4SS | 3.0.1 (experimental build 1164) | reflection dump to generate the usmap | only for generating mappings; not shipped |
| Mappings (usmap) | generated from the running game for the tested build | unversioned-property parsing | required by CUE4Parse for this game |
| io_scene_psk_psa | 9.1.3 Blender extension | PSK/PSA import | for umodel exports (not needed with native extractor) |
| FFVII-Rebirth-Mesh-Patcher | 1.7.2 | classifies mesh as new/old format | check-only use; a freshly cooked mesh was "new format" |

## Environment rules

1. **Dedicated editor prefix.** Create a Proton prefix just for the editor, with env: `STEAM_COMPAT_DATA_PATH=<editor-prefix-parent>`, `STEAM_COMPAT_CLIENT_INSTALL_PATH=<steam-root>`, `SteamAppId=0`, `SteamGameId=0`, `WINEDEBUG=fixme-all`, `PROTON_LOG=1`; unset `WINEPREFIX`, `PYTHONPATH`, `PYTHONHOME` before running. Verify Proton `version` file equals the pinned string before every run. [OBSERVED]
2. **Throttle** the editor: `nice -n 10` and `taskset` to a few cores so the desktop stays usable. [DECISION]
3. **GPU/VRAM.** The editor ran `-NullRHI` (no GPU). The game needs VRAM headroom: ~4 GiB free was judged insufficient; owner stopped a local LLM service during game tests. [OBSERVED] Do not stop unrelated services without the owner's say-so.
4. **Display.** The game window needs the host's X11 socket (`/tmp/.X11-unix/X0`) through Xwayland. When that vanished, the game started but had no window ("no window driver"), and the editor under Proton hung (600 s import timeout, no log). Workaround for the editor: a separate authenticated Xwayland instance (`:99`) used only for the editor. Game: owner re-login restored the socket. See P13/P14. [OBSERVED]
5. **Hash-pin everything**: tool binaries, editor project file, Proton version, extracted metadata, source scene. Every stage re-verifies pins (`require_hash`).
6. **No downloads at runtime**: Oodle must be initialised from an explicit local path (never the auto-download overload); AES keys, if the build needs them, come from a user-authorised key file and are never logged or stored. [OBSERVED design]
7. **Layout** (suggested): `build/` immutable outputs; `tools/`; `config/` pins; `validation/private/` reports/screens; `backups/`; `install_manifests/`. Keep game data, mods, saves, keys out of VCS (private repo ≠ backup).

## Pre-flight smoke (run once, record outputs)

* Blender: run a headless script that prints version, bundled FBX operator availability, add-ons; no network. 
* Editor: run a trivial commandlet-style python script that logs engine version and exits; require version string match.
* Extractor: list one container's entries with mappings loaded.
* Steam inventory: read-only snapshot of game dir, build id, exe hash, installed mods, processes.
