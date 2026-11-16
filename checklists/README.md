# Checklists

Copy a checklist into your work log, tick items **only with evidence** (report path, measured value, screenshot path). An unticked item is `NOT_RUN`, not "probably fine". Status vocabulary: `PASS`, `FAIL`, `NOT_RUN`, `BLOCKED`, `NEEDS_ART_REVIEW` ([docs/01](../docs/01-evidence-discipline.md)).

Line format to use when ticking:

```
- [x] <item> — evidence: <report/file path>; value: <measured>; tag: [OBSERVED]
```

| File | Use at |
|---|---|
| [00-session-start.md](00-session-start.md) | beginning of every session (new or resumed) |
| [01-toolchain-smoke.md](01-toolchain-smoke.md) | first setup, and after any tool/game version change |
| [02-art-approval.md](02-art-approval.md) | before the heavy pipeline touches a scene |
| [03-blender-source-qa.md](03-blender-source-qa.md) | every Blender revision and game-source prep |
| [04-fbx-and-editor-import.md](04-fbx-and-editor-import.md) | FBX export, patch, editor import, saved-bind audit |
| [05-materials.md](05-materials.md) | bake, translate, native-parent instances, container parameter check |
| [06-runtime-and-animation.md](06-runtime-and-animation.md) | runtime attachments and (optional) animation |
| [07-cook-package-inspect.md](07-cook-package-inspect.md) | cook, IoStore, inspection |
| [08-install-rollback.md](08-install-rollback.md) | before/after any mutation of the game directory |
| [09-in-game-test.md](09-in-game-test.md) | every launch of the game for verification |
| [10-delegation-brief.md](10-delegation-brief.md) | writing a task for a sub-agent |
| [11-session-end.md](11-session-end.md) | end of every session |
| [12-privacy-and-publication.md](12-privacy-and-publication.md) | before committing/pushing anything |
