# Rules.md: Project Rules

*What to use. What to avoid. No exceptions.*

## What to Use

- **Ownership:** edit files only inside your own folders. Need something from another folder? Call its public function or ask the owner.
- **Contracts:** modules talk only through the JSON shapes and function signatures in Architecture.md and `/shared/schemas`.
- **Mock-first:** develop against `/shared/samples` (mock_vector.json, mock_scale.json, mock_report.json, mock.glb) until the real module arrives.
- **Units:** pixels before scaling, metres after. Always name variables with the unit (`length_px`, `length_m`).
- **Config:** every threshold and constant lives in the owner's `config.py`, never inline.
- **Python:** type hints, snake_case, small functions, docstring with input/output shape. **TypeScript:** strict mode, camelCase, one component per file.
- **Reproducibility:** fixed random seed (42), model version string in every result, same preprocessing at train and inference time.
- **Commits:** prefix with the area, for example `[ml] add dice loss`, `[geo] snap endpoints`, `[app] rooms table`.
- **Evaluation:** test only on held-out plans never used for training or threshold tuning.

## What to Avoid

- Do not edit another person's folder, even for a "tiny fix". Message them instead.
- Do not rename or remove contract fields. Adding an optional field needs a group message first.
- Do not commit datasets, model weights or outputs (use Drive / Hugging Face Hub).
- Do not use `git push --force`, and do not push directly to main.
- Do not add a new library without telling the group and updating Architecture.md.
- Do not hide uncertainty: low-confidence or guessed geometry must be flagged in warnings, never silently invented.
- Do not tune thresholds on the held-out test set.
- Do not use browser storage in the viewer; keep state in React state.
- Do not leave print-debugging or commented-out code in main.
- Do not ask the AI tool to "build the whole app". One phase, one module, one prompt at a time.

> **Tip:** Re-upload this document whenever a new AI session starts. It re-anchors the tool instead of letting it drift.
