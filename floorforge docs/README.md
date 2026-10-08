# FloorForge

**Floor plan image → correctly scaled, navigable 3D model.**
Built for HackNex 2026, problem statement **HNX26EPS06** (3D Scene Generation from Blueprints and Room Video), **Mode A: floor plan → 3D**.

A trained segmentation model finds walls, doors, windows and rooms. Geometry code cleans them into real wall segments, printed dimension labels are read with OCR to recover true scale, a validator checks the layout, and a 3D builder exports a GLB that opens in a browser viewer.

> Status: hackathon prototype. Numbers in the results table are filled in from our own held-out set, not from judges' data.

---

## What it does

1. Upload a floor-plan image (scan, photo, CAD export).
2. **Segmentation** (U-Net, ResNet34 encoder): wall / door / window / room masks.
3. **Vectorization:** clean wall segments snapped to right angles, doors and windows attached to walls, room polygons.
4. **Scale recovery:** OCR of dimension labels with a robust solver. Falls back to a door-width prior, clearly labelled "estimated". Manual override available.
5. **Validation:** closed rooms, openings on walls, plausible sizes. Problems appear as warnings, never hidden.
6. **3D model:** walls with thickness and openings, floors, exported as GLB.
7. **Viewer:** orbit, plan overlay, room-dimensions table, scale badge, download.

## Pipeline

```
plan image
  -> ml.infer.predict()                 [Person A]  SegResult (mask + confidence)
  -> geometry.vectorize.vectorize()     [Person B]  FloorPlanVector (pixels)
  -> scale.solve.solve_scale()          [Person A]  ScaleResult
  -> geometry.build.build_model()       [Person B]  model.glb + PlanReport (metres)
  -> backend /api/reconstruct           [Person C]
  -> React + Three.js viewer            [Person C]
```

## Repository layout

```
floorforge/
  shared/        JSON schemas + mock sample files (contracts only)
  ml/            data prep, training, inference           (Person A)
  scale/         OCR + scale solver                       (Person A)
  eval/          metrics, baseline, ablation script       (Person A)
  geometry/      vectorize, snap, validate, build GLB     (Person B)
  backend/       FastAPI app and pipeline                 (Person C)
  frontend/      React + Vite + Three.js app              (Person C)
  docs/          PRD, Architecture, Rules, Phases, Design, Memory, write-up
  data/ models/ outputs/    git-ignored
```

## Setup

Requirements: Python 3.11, Node 18+, a GPU for training (Colab T4 is enough).

```bash
git clone <repo-url> floorforge && cd floorforge
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate

pip install -r ml/requirements.txt
pip install -r geometry/requirements.txt
pip install -r backend/requirements.txt

cd frontend && npm install && cd ..
```

Model weights are not stored in git. Download `best.pt` from the team Drive / Hugging Face Hub into `models/best.pt`, or set `MODEL_PATH`:

```bash
export MODEL_PATH=models/best.pt
```

## Run

```bash
# backend (from repo root)
uvicorn backend.main:app --reload --port 8000

# frontend
cd frontend && npm run dev          # http://localhost:5173
```

Train the model (Colab or local GPU):

```bash
python -m ml.data_prep              # CubiCasa5K SVG -> 5-class masks
python -m ml.train                  # saves models/best.pt
```

Evaluate and generate the ablation table:

```bash
python -m eval.run_ablation         # writes eval/results/ablation.md
```

## API

| Endpoint | Purpose |
|---|---|
| `POST /api/reconstruct` | multipart `file`, optional `ppm_override`; returns PlanReport + `job_id`, `overlay_url`, `glb_url`, `plan_url` |
| `GET /api/files/{job_id}/{name}` | serves `model.glb`, `overlay.png`, `vector.json` |
| `GET /api/samples` | sample plans for the upload screen |
| `GET /api/health` | model version and status |

Units: pixels before scaling (x right, y down), metres after. 3D scene: Y up, X = x_px / ppm, Z = y_px / ppm.

## Data

- **Training:** CubiCasa5K (about 5,000 annotated floor plans). Check its licence before any use beyond the hackathon.
- **Evaluation:** our own held-out set of 30–40 plans in varied styles with real dimensions, never used for training or tuning.

## Results

Filled from `eval/results/ablation.md`.

| Variant | Layout IoU | Dim. error % | Opening F1 |
|---|---|---|---|
| Baseline parser | | | |
| U-Net + plain extrusion | | | |
| + style augmentation | | | |
| + line snapping | | | |
| + OCR scale solver | | | |
| + topology validator (full) | | | |

## Team workflow

- Three people, strict folder ownership (see `docs/Rules.md`). Never edit a folder you do not own.
- Branches: `a/ml`, `b/geometry`, `c/app`. `main` is protected. Merge only at integration windows (hours 6, 12, 18, 22), in order A, B, C.
- `git add` only your own folders. No force-push.
- Develop against `shared/samples` mocks until the real module is merged.
- Contract changes (`shared/schemas`) need agreement from all three.

## Limitations

- Accuracy depends on how similar a plan's style is to the training data.
- Without printed dimensions, scale is estimated and marked as such.
- Static single-floor plans only. No furniture, no room-video input.

## Documentation

`docs/PRD.md` · `docs/Architecture.md` · `docs/Rules.md` · `docs/Phases.md` · `docs/Design.md` · `docs/Memory.md`

## Licence

To be decided by the team. Dataset and model licences apply separately.
