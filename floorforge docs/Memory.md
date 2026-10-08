# Memory.md: Project Memory

*The living document that keeps your AI tool caught up.*

## How to keep it

- Append after every major session: what was built, what changed, and why. Re-upload at the start of the next session.
- One entry per decision. Never delete old entries; mark them "superseded" instead.
- Entry template: **ID · date/hour · decision · reason · who owns it.**

## Decisions so far

| ID | Decision | Reason | Owner |
|---|---|---|---|
| D-001 | Chose HNX26EPS06 (3D from blueprints), Mode A: floor plan → 3D | Best fit: public data, trainable model, strong demo, metric-friendly rubric | All |
| D-002 | Model: U-Net with ResNet34 encoder, classes bg / wall / door / window / room | Thin structures need skip connections; fast on a T4 | A |
| D-003 | Dataset: CubiCasa5K (verify license), plus 30–40 own held-out plans in varied styles | Public, large, labelled; own set tests style shift | A |
| D-004 | Scale: OCR of dimension labels + robust solver; door-width fallback marked "estimated"; manual override | Dimension error is part of the 40% layout score | A |
| D-005 | Geometry: mask + classical line detection, snap to right angles, validator, walls as box pieces with openings, export GLB | Clean, correctly scaled output; viewer-friendly | B |
| D-006 | Team of 3 with strict folder ownership and JSON contracts | Prevents overlapping or overwriting code | All |
| D-007 | Research contribution = ablation table (baseline, + augmentation, + snapping, + OCR scale, + validator) | 20% of rubric; shows what is new | A |

## Open questions

- Which baseline do we run (classical parser, an open-source floor-plan parser, or both)? Owner: A.
- Does EasyOCR or PaddleOCR read dimension text better on our plans? Owner: A.
- Does the CubiCasa5K licence allow our use? (verify) Owner: A.
- Final viewer extras (measure tool, confidence colouring) depend on Phase 3 progress. Owner: C.

## Session log

| Date / hour | Who | Built / changed | Next |
|---|---|---|---|
| Kickoff | All | Documents created, contracts agreed | Phase 0 setup |
|  |  |  |  |
