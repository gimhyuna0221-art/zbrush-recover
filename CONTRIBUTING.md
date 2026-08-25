# Contributing

Recovery behavior varies across ZBrush versions, so new evidence is useful.

Please open an issue or pull request with:
- exact ZBrush version,
- exact error text,
- whether the file is ZPR/ZTL/QuickSave/Recovered,
- whether `Tool > SubTool` is accessible,
- relevant `LogCorruptedFiles.txt` lines,
- the recovery step that succeeded or failed.

Do not commit proprietary or client model files without explicit permission.

When documenting binary structures, distinguish:
- confirmed observations,
- hypotheses,
- version-specific offsets.

Never generalize a hard-coded byte offset from one ZBrush file to all files.
