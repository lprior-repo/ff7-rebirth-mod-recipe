# 12 — Privacy and publication (before any commit or push)

Version control in a modding project is for **process**, never assets. A private repository is not a backup and not a licence to store game data.

## Must stay out of version control
- [ ] Game extractions, mappings, keys, tokens, credentials.
- [ ] Models, textures, renders of copyrighted/third-party content (unless the human explicitly says otherwise for specific files), FBX/PSK, `.blend` scenes.
- [ ] Editor project, cooked outputs, containers, installers, third-party mod archives.
- [ ] Saves, config, Steam/Proton prefixes, inventories, receipts, backups, crash dumps.
- [ ] Absolute home paths, usernames, machine names, account names.
- [ ] Hashes that identify specific private assets (keep only in the private ledger).

## Scan before commit
- [ ] Search the tree for: home-directory prefix, username, 64-hex strings, 8-hex candidate-like ids, mod-author/owner names, file extensions of assets (`.blend .fbx .psk .uasset .ubulk .ucas .utoc .pak .png .jpg .usmap`).
- [ ] `.gitignore` covers all of the above categories **and** was in place before the first `git add`.
- [ ] Staged file list reviewed by eye.
- [ ] Remote visibility confirmed as intended (private unless the human decided otherwise).

## Publication
- [ ] Third-party licences checked for anything redistributed (tools, loaders, bases).
- [ ] No DRM bypass guidance; no redistribution of game data.
- [ ] External uploads (to any third-party service) only with the human's explicit permission for that specific upload.
