---
name: zbrush-recover
description: Diagnose and salvage corrupted ZBrush QuickSave, Recovered ZPR, and ZTL files while preserving originals. Use when ZBrush crashes while saving, a recovered project crashes on load, or a corrupted SubTool prevents a Tool from opening normally.
---

# zbrush-recover

## Purpose
Recover as much usable ZBrush geometry/SubTool data as possible from a corrupted QuickSave, Recovered ZPR, or ZTL while minimizing further damage.

This skill is designed for cases such as:
- ZBrush crashes while QuickSave/AutoSave is running.
- A `Recovered_####.zpr` exists but opening it crashes ZBrush.
- `File > Open` and/or `Tool > Load Tools From Project` crash.
- A recovered ZTL loads with `SubTool ... is corrupted`.
- One corrupted SubTool prevents otherwise healthy SubTools from being used.

## Core rules

1. Never overwrite the only surviving recovery file.
2. Work only on copies.
3. Do not claim a file is unrecoverable until the supported recovery branches have been exhausted.
4. Treat ZBrush version as part of the file format. Record the exact version before binary-level recovery.
5. Prefer native ZBrush recovery paths before binary surgery.
6. If ZBrush loads a Tool far enough to show a SubTool list, prioritize salvaging healthy SubTools over repairing the whole project.
7. After successful recovery, immediately save a new ZTL, reopen-test it, and create a second backup.

## Intake

Collect:
- Exact ZBrush version, e.g. `2023.2.2`.
- Corrupted file type and name.
- Whether the crash happened during QuickSave/AutoSave.
- Whether there is an older healthy ZPR/ZTL for comparison.
- Exact error message or Windows Event Viewer crash code when useful.
- `LogCorruptedFiles.txt` when ZBrush creates it.

For ZBrush 2023, a common data location is:
`C:\Users\Public\Documents\ZBrushData2023\`

Exact recovery folders can vary by ZBrush version and configuration.

Common recovery locations:
- `QuickSave`
- `RecoveredFiles` (if present; location and availability vary by ZBrush version)
- `LogCorruptedFiles.txt`
- `ZPluginData\TransposeMasterData`

Do not assume these folders contain the missing work. Confirm what stage of the workflow was active when the crash occurred.

## Stage 0 — Preserve evidence

Before testing:
- Copy the corrupted ZPR/ZTL to a separate folder.
- Preserve the entire QuickSave/RecoveredFiles folder if available.
- Do not resave the corrupted original.
- Do not delete `LogCorruptedFiles.txt`.
- Do not run repeated tests against the sole original.

## Stage 1 — Native ZBrush recovery

Try in this order.

### A. Undo History bypass
In a clean ZBrush session:
`Preferences > Undo History > Skip Loading`

Then:
`File > Open`

Use only as a diagnostic/recovery attempt. If it crashes, move on.

### B. Extract Tools from Project
In a clean ZBrush session:
`Tool > Load Tools From Project`

Select the damaged ZPR.

If a Tool loads:
- Save it immediately as a new `.ZTL`.
- Do not overwrite the damaged source.

If this crashes too, proceed to Stage 2.

## Stage 2 — Standalone ZTool extraction

Use when:
- ZPR project loading crashes,
- `Load Tools From Project` crashes,
- but the ZPR contains recognizable ZTool records.

Goal:
Extract the recovered root `ZTool` record from the damaged ZPR and rewrap it as a standalone `.ZTL` using a healthy ZTL from the same ZBrush version as the structural wrapper.

Important:
- Preserve the recovered ZTool payload unchanged whenever possible.
- Use a same-version healthy ZTL as the wrapper/template.
- Do not invent offsets or lengths; derive and validate them from the actual files.
- Verify resulting container lengths before asking the user to open it.

### Proven ZBrush 2023.2.2 success pattern
A standalone recovered ZTL may produce:

`File has been loaded successfully, however SubTool "...Recovered_Tool" was detected as corrupted.`

This is not necessarily total failure.

If this message appears:
1. Click `OK`.
2. Do NOT draw the Tool onto the canvas.
3. Open `Tool > SubTool`.
4. Inspect whether multiple SubTools are listed.
5. If one clearly corrupted placeholder/root SubTool is present, especially `Recovered_Tool`, do not click/draw it.
6. Delete only that corrupted SubTool.
7. Verify the remaining SubTools.
8. Save immediately as a new ZTL.
9. Close ZBrush and reopen the new ZTL to verify persistence.

### Real confirmed recovery result
In one ZBrush 2023.2.2 case:
- Original Recovered ZPR crashed on open.
- `Load Tools From Project` also crashed.
- The recovered root ZTool was rewrapped as a standalone ZTL.
- ZBrush loaded the ZTL and warned that `Recovered_Tool` was corrupted.
- The SubTool list was still accessible.
- Deleting the corrupted `Recovered_Tool` SubTool caused the remaining model data to become usable.
- The user's lost modeling work was successfully recovered.

This is the preferred path when the warning names a single corrupted `Recovered_Tool` but the rest of the SubTool list is visible.

## Stage 3 — SubTool salvage

If the recovered ZTL loads and multiple SubTools are listed:

1. Do not immediately draw the full Tool.
2. Avoid clicking a SubTool known to crash ZBrush.
3. Test healthy-looking SubTools one by one.
4. Export critical survivors individually if needed:
   - OBJ for geometry-first salvage.
   - Then save a clean ZTL after removing corrupted entries.
5. If one SubTool crashes ZBrush, skip it and reopen the recovered Tool to continue testing others.

Priority:
Preserve geometry first. Project state, Undo history, materials, and plugin state are secondary.

## Stage 4 — Corruption logs

Check:
`C:\Users\Public\Documents\ZBrushData2023\LogCorruptedFiles.txt`

Examples:
- `SubTool "HAND_POSITION" was loaded but detected as corrupted.`
- `SubTool "...TransposeMasterData\CHAR_..." was loaded but detected as corrupted.`
- `SubTool "...QuickSave//Recovered_Tool" was loaded but detected as corrupted.`

Interpretation:
- A named SubTool corruption message can localize damage.
- A single corrupt SubTool does not prove the entire Tool is unusable.
- Repeated generic `Unable to read a subtool` warnings after binary patching may indicate the patch itself broke serialized SubTool boundaries. Stop using that patch rather than concluding all SubTools are corrupt.

## Stage 5 — Compare against an older healthy file

If an older healthy ZTL/ZPR exists:
- Compare it byte-for-byte with the recovered file.
- Identify which records are unchanged, expanded, new, or structurally inconsistent.
- Use this comparison to distinguish:
  - old baseline data,
  - newly written post-baseline data,
  - plugin/Transpose Master bookkeeping,
  - actual mesh/SubTool changes.

Do not assume a larger record automatically means newer geometry. Confirm with ZBrush behavior.

## Binary recovery cautions

Binary surgery is a last resort.

Do:
- Keep original files immutable.
- Validate headers, record lengths, offsets, and end boundaries.
- Use a healthy same-version file as a structural reference.
- Change the fewest bytes possible.
- Generate separate recovery candidates instead of repeatedly mutating one file.

Do not:
- Guess ZBrush container offsets.
- Infer SubTool count from filenames alone.
- Treat every `UndoCounter` string as usable Undo History.
- Assume Transpose Master records contain the user's later modeling simply because they are newer.
- Continue patching after a test generates dozens of cascading `Unable to read a subtool` warnings; that usually indicates serialization boundaries were broken.

## Decision tree

### Case 1
Damaged ZPR opens normally.
→ Save As immediately to a new ZPR/ZTL.

### Case 2
`File > Open` fails, but `Load Tools From Project` works.
→ Save recovered Tool as new ZTL.

### Case 3
Both native loading methods crash.
→ Extract root ZTool and build standalone same-version ZTL.

### Case 4
Standalone ZTL says:
`loaded successfully, however SubTool ... is corrupted`
and SubTool list is visible.
→ Do not draw.
→ Delete only the corrupt SubTool.
→ Save surviving Tool as new ZTL.
This is the key proven recovery route.

### Case 5
Healthy SubTools can be selected but one crashes ZBrush.
→ Reopen recovered ZTL.
→ Skip the bad SubTool.
→ Export/save healthy SubTools individually.

### Case 6
Standalone ZTL cannot be parsed at all.
→ Compare against an older healthy same-version file and perform record-level analysis.

## Completion checklist

Recovery is complete only after:
- A clean new `.ZTL` has been saved.
- The clean ZTL closes and reopens successfully.
- Important SubTools are present.
- A second backup exists in another folder/drive.
- Optional: critical geometry is also exported to OBJ/FBX.
- The corrupted original is retained until the user confirms the recovered project is complete.

## Prevention after recovery

For active modeling:
- Use incremental saves: `v01`, `v02`, `v03`...
- Do not rely on QuickSave as the only copy.
- Keep at least one manual ZTL/ZPR checkpoint before large Transpose Master, topology, or proportion changes.
- Periodically reopen a saved checkpoint to verify it is actually readable.
