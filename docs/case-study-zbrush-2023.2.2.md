# Case Study — ZBrush 2023.2.2 QuickSave Recovery

## Status

Confirmed successful recovery.

## Failure scenario

- ZBrush version: **2023.2.2**
- The artist had performed substantial modeling after the last manual save.
- ZBrush crashed while QuickSave/recovery data was being written.
- Only the recovered project contained the later session state.
- Opening the recovered ZPR caused ZBrush to terminate.
- `Tool > Load Tools From Project` also caused a crash.
- `Preferences > Undo History > Skip Loading` did not solve the problem.

## Misleading paths

Several observations were useful diagnostically but were **not sufficient evidence** of recoverable geometry:

- Transpose Master record names with later suffixes
- strings such as `UndoCounter`
- larger binary regions by themselves

The recovery process therefore avoided treating these as proof of the missing modeling state.

## Breakthrough

The recovered root ZTool was isolated from the damaged ZPR and wrapped as a standalone ZTL compatible with the same ZBrush version.

ZBrush then displayed:

```text
File has been loaded successfully, however SubTool
".../QuickSave//Recovered_Tool"
was loaded but detected as corrupted.
```

Crucially, ZBrush remained open.

Attempting to draw/use the complete Tool still crashed ZBrush, but the **SubTool list was visible**.

## Successful recovery

The artist:

1. Accepted the corruption warning.
2. Did **not** draw the recovered Tool onto the canvas.
3. Opened `Tool > SubTool`.
4. Found multiple SubTools still listed.
5. Deleted the corrupted `Recovered_Tool` entry.
6. The remaining SubTools became usable.
7. Saved the recovered result under a new filename.

The missing modeling work was recovered.

## Generalizable lesson

A recovered ZPR/ZTL that crashes when used may still contain healthy SubTools.

When ZBrush can parse enough of the Tool to show a SubTool list, the best recovery strategy may be to **salvage healthy SubTools and remove the corrupt entry**, rather than trying to repair the entire project container.

## Limits

This does not imply that every corrupt ZBrush file is recoverable. A file may be truncated, have multiple damaged mesh records, or fail before ZBrush can enumerate SubTools.

Binary-level repair must be derived from the specific file and exact ZBrush version. Never copy hard-coded offsets from this case into another file.
