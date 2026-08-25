# ZBrush Recover

An Agent Skill for diagnosing and salvaging corrupted **ZBrush QuickSave / Recovered ZPR / ZTL** files.

The workflow was developed from a real recovery case where ZBrush 2023.2.2 crashed during QuickSave, the resulting recovered project crashed on load, and the usable modeling work was ultimately recovered by isolating the recovered ZTool, opening it as a standalone ZTL, entering the SubTool list without drawing the Tool, and deleting the single corrupted `Recovered_Tool` SubTool.

> This is a recovery workflow for AI coding/desktop agents. It is **not** a Maxon product and is not affiliated with Maxon or ZBrush.

## Install

This repository follows the open `SKILL.md` Agent Skills format.

### Any supported agent via the `skills` CLI

```bash
npx skills add gimhyuna0221-art/zbrush-recover
```

Install globally:

```bash
npx skills add gimhyuna0221-art/zbrush-recover --skill zbrush-recover -g
```

### Claude Code

```bash
npx skills add gimhyuna0221-art/zbrush-recover --skill zbrush-recover -a claude-code -g
```

### Codex

```bash
npx skills add gimhyuna0221-art/zbrush-recover --skill zbrush-recover -a codex -g
```

The `skills` CLI is an independent open-source installer that supports Claude Code, Codex, and many other agents.

## What the skill does

It guides an agent through a conservative recovery ladder:

1. Preserve the only surviving file and work on copies.
2. Record the exact ZBrush version.
3. Try native ZBrush recovery paths first.
4. Inspect `LogCorruptedFiles.txt` when available.
5. If project loading still crashes, inspect/extract a recoverable ZTool rather than repeatedly opening the broken ZPR.
6. If a standalone recovered ZTL loads but reports one corrupted SubTool, avoid drawing the Tool, enter `Tool > SubTool`, isolate/delete the corrupted entry, and salvage the healthy SubTools.
7. Use binary-level inspection only as a last resort and validate offsets/record lengths against a healthy file from the same ZBrush version.
8. Save a clean new ZTL and verify that it closes and reopens.

## Confirmed recovery pattern: ZBrush 2023.2.2

A real case followed this path:

```text
QuickSave crash
    ↓
Recovered_####.zpr exists
    ↓
File > Open → ZBrush crashes
    ↓
Tool > Load Tools From Project → ZBrush crashes
    ↓
Recovered root ZTool extracted/re-wrapped as standalone ZTL
    ↓
ZBrush: "File has been loaded successfully,
however SubTool ... Recovered_Tool is corrupted."
    ↓
Do NOT draw the Tool
    ↓
Tool > SubTool
    ↓
Delete the corrupted Recovered_Tool SubTool
    ↓
Remaining SubTools become usable
    ↓
Save as a new ZTL and reopen-test
```

The important lesson is that a single corrupted SubTool can make the whole recovered Tool appear unusable even when other SubTools are still present.

See [the case study](docs/case-study-zbrush-2023.2.2.md).

## Safety rules

- Never overwrite the only surviving recovery file.
- Perform experiments on copies.
- Do not treat a bigger binary record as proof that it contains newer geometry.
- Do not assume `UndoCounter` strings are usable Undo History.
- Do not assume newer Transpose Master records contain later freeform modeling.
- If an experimental binary patch suddenly produces dozens of cascading `Unable to read a subtool` errors, stop: the patch may have broken serialization boundaries.
- Keep the original damaged file until the recovered model has been saved, closed, and reopened successfully.

## Compatibility

The diagnostic workflow is intended to be version-sensitive.

**Confirmed:** ZBrush 2023.2.2 recovery case described above.

Other versions may use different binary structures or recovery behavior. The skill tells the agent to identify the exact version and avoid reusing hard-coded offsets from another file/version.

## Repository layout

```text
zbrush-recover/
├─ README.md
├─ LICENSE
├─ CHANGELOG.md
├─ CONTRIBUTING.md
├─ SECURITY.md
├─ docs/
│  └─ case-study-zbrush-2023.2.2.md
└─ skills/
   └─ zbrush-recover/
      └─ SKILL.md
```

## Reporting another recovery case

If the workflow helps — or fails — please open an issue with:

- ZBrush version
- what operation was running when ZBrush crashed
- file type (`.zpr`, `.ztl`, QuickSave, Recovered file)
- exact ZBrush error text
- relevant lines from `LogCorruptedFiles.txt`
- whether the SubTool list was visible

**Do not upload commercial/proprietary models unless you have permission to share them.**

## License

MIT. See [LICENSE](LICENSE).
