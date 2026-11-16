# 08 — Install and rollback

## Preconditions (every time)
- [ ] Human approved this install (what, where, which candidate).
- [ ] **Fresh** inventory this session; game executable hash == inventory; `release.game_build_id` == installed build id.
- [ ] Game and Steam-launched game processes **not running** (process check, not memory).
- [ ] Release inspection report is PASS for the **same release hash**.
- [ ] Backup exists of saves, game config, existing mod files, compat/launch settings; backup hashed; backup location recorded privately.
- [ ] Only one custom candidate will be installed; others from earlier receipts are rolled back first.
- [ ] Required third-party plugins (if the release declares any) are installed.

## Manifest validation (refuse on doubt)
- [ ] `stage` in the allowed list; sources relative/canonical/no symlinks; destinations only under approved dirs.
- [ ] No case-insensitive destination collisions; destinations do not already exist.
- [ ] File hashes match; container sets complete (`.utoc` ⇄ `.ucas`).

## Install
- [ ] Transaction dir (private mode) + empty receipt path; refuse if they exist.
- [ ] Files staged and re-hashed; hard-linked into place (no overwrite); directories created are recorded.
- [ ] Installed bytes re-hashed; mismatch ⇒ roll back what this receipt created.
- [ ] Receipt `INSTALLED`, `gameplay_verified: false`.
- [ ] Receipt path recorded in the ledger as the **exact** current receipt.

## Rollback (by receipt only)
- [ ] Receipt status not already `ROLLED_BACK`; game not running.
- [ ] Every owned file: device/inode and hash equal the receipt; otherwise **abort and report "user-modified file preserved"**.
- [ ] Files moved to quarantine (not deleted); quarantine verified; only receipt-created empty dirs removed.
- [ ] Receipt status `ROLLED_BACK`; ledger updated; failed candidate and crash dumps kept.
- [ ] Post-rollback inventory shows no owned files; unrelated mods/saves/config untouched.
- [ ] If the game is relaunched afterwards, observe the stock look and record it.
