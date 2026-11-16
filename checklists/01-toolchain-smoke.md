# 01 — Toolchain smoke (record versions; run each tool once on a trivial input)

## Pins
- [ ] Game: Steam app id, **build id**, game executable hash recorded (read-only).
- [ ] Blender version string + bundled Python version recorded; scripts that import `bpy` run under Blender's Python only.
- [ ] Host Python version + venv; no `bpy` expected there.
- [ ] .NET SDK version; `DOTNET_GENERATE_ASPNET_CERTIFICATE=false` set **before the first dotnet run** (P15).
- [ ] Extractor library version(s) + dependency pins (note advisories/licences, [docs/19](../docs/19-sources.md)).
- [ ] Container converter commit + patch recorded; confirm it handles the game's container header (P4).
- [ ] Editor project/engine version string recorded (`4.26.2-…` expected); project file hash.
- [ ] Proton version; **dedicated prefix** for the editor (never the game's prefix).
- [ ] Mapping (usmap) generated from the *running build*; build id recorded in the file's provenance.

## Smoke runs (each into a fresh output)
- [ ] Blender headless prints version, FBX exporter available, no network use, autoexec policy known.
- [ ] Extractor lists one container's entries with mappings loaded; Oodle from an explicit local path (no auto-download).
- [ ] Extractor decodes the target native mesh: **8-influence skin and all LODs** present (P2); bone/socket/slot counts recorded.
- [ ] Converter converts one native skeleton; result opens in the editor.
- [ ] Editor python smoke logs engine version and exits within timeout; exit code AND report file checked (P1).
- [ ] Display provisioned for the editor (X socket present or dedicated authenticated server) (P13/P14).
- [ ] CPU throttle for the editor (nice/taskset) applied.
- [ ] GPU/VRAM headroom for the game measured; which service, if any, may be stopped for tests (human decision) recorded.
- [ ] Steam inventory snapshot created and stored privately.

## Pass
- [ ] All pins written to a lock file; every later stage re-verifies pins by hash.
- [ ] No keys, extractions, or game data are inside version control (privacy checklist 12).
