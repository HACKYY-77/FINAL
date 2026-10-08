# Architecture.md: App Flow & Architecture

*How everything actually connects and where it lives.*

## App Flow, Screen by Screen

| Step | User action | What happens next |
|---|---|---|
| Screen 1: Upload | Drag-drop an image, or click a sample plan, then press **Reconstruct**. | Frontend POSTs the file to `/api/reconstruct` and shows Screen 2 in loading state. |
| Loading | Waits (typically seconds). | Progress labels: Segmenting → Vectorizing → Reading dimensions → Validating → Building 3D. |
| Screen 2: Result | Orbits the 3D model; toggles overlay (mask / vector / confidence). | Left: plan with overlay. Right: 3D viewer. Sidebar: rooms table, scale badge, warnings, download. |
| Scale override | Types a known length or pixels-per-metre value. | Frontend re-POSTs with `ppm_override`; backend re-runs only scale + build. |
| Download | Clicks Download GLB. | Browser fetches `/api/files/{job_id}/model.glb`. |
| Error | Unreadable image or no walls found. | Friendly message with a retry button; never a blank screen. |

## Pipeline and Data Flow

```
plan image
   |  ml.infer.predict()                       [Person A]
   v
SegResult: mask (0 bg,1 wall,2 door,3 window,4 room) + confidence map
   |  geometry.vectorize.vectorize()           [Person B]
   v
FloorPlanVector (pixel units)
   |  scale.solve.solve_scale()                [Person A]
   v
ScaleResult (pixels_per_meter, method, matches)
   |  geometry.build.build_model()             [Person B]  (runs validator inside)
   v
model.glb + PlanReport (metres)
   |  backend /api/reconstruct                 [Person C]
   v
React + Three.js viewer                        [Person C]
```

## Data Contracts (the only things modules share)

Pixel stage: x right, y down. After scaling: metres. 3D scene: Y is up, X = x_px / ppm, Z = y_px / ppm. Field names never change without all three people agreeing.

### FloorPlanVector (JSON, pixel units)

```json
{ "image": {"width": 1024, "height": 768},
  "walls":    [{"id":"w1","p1":[0,0],"p2":[100,0],"thickness_px":12.0,"confidence":0.93}],
  "openings": [{"id":"o1","type":"door|window","wall_id":"w1",
                "offset_px":120.0,"width_px":38.0,"confidence":0.88}],
  "rooms":    [{"id":"r1","polygon_px":[[0,0],[100,0],[100,80],[0,80]],"label":null,"confidence":0.90}] }
```

### ScaleResult (JSON)

```json
{ "pixels_per_meter": 52.4, "method": "ocr|door_prior|manual", "confidence": 0.8,
  "matches": [{"text":"3.50 m","value_m":3.5,"wall_id":"w4","ratio":51.9}] }
```

### PlanReport (JSON, metres)

```json
{ "glb_path": "outputs/<job_id>/model.glb",
  "scale": { "...ScaleResult...": "..." },
  "rooms": [{"id":"r1","label":"Bedroom","width_m":3.5,"length_m":4.1,"area_m2":14.3,"confidence":0.9}],
  "walls": [{"id":"w1","length_m":4.1,"thickness_m":0.15,"confidence":0.93}],
  "warnings": [{"code":"OPENING_OFF_WALL","element_id":"o3","message":"..."}] }
```

## Backend API (FastAPI)

| Endpoint | Purpose |
|---|---|
| `POST /api/reconstruct` | multipart: `file`, optional `ppm_override`. Returns PlanReport plus `job_id`, `overlay_url`, `glb_url`, `plan_url`. |
| `GET /api/files/{job_id}/{name}` | Serves `model.glb`, `overlay.png`, `vector.json` for a job. |
| `GET /api/samples` | Lists sample plans for the upload screen. |
| `GET /api/health` | Returns model version and status. |

## File and Folder Structure

```
floorforge/
  shared/        contracts only: schemas/, samples/ (mock files)   changes by agreement
  ml/            data prep, training, infer.py, config.py          Person A
  scale/         OCR + scale solver, config.py                     Person A
  eval/          metrics, baseline, ablation script                Person A
  geometry/      vectorize.py, snap.py, validate.py, build.py      Person B
  backend/       FastAPI app, orchestration, job storage           Person C
  frontend/      React + Vite + Three.js app                       Person C
  docs/          these six documents, write-up                     Person C
  data/  models/  outputs/     git-ignored (weights shared via Drive/HF Hub)
```

## Tech Stack

| Layer | Choice |
|---|---|
| Language | Python 3.11 (backend/ML), TypeScript (frontend) |
| ML | PyTorch, segmentation_models_pytorch (U-Net, ResNet34 encoder), albumentations, Colab T4 for training |
| Vision / geometry | OpenCV, scikit-image, NumPy, SciPy (least squares), Shapely, trimesh (+ earcut) for GLB |
| OCR | EasyOCR (simple install); PaddleOCR as optional alternative (verify which reads dimension text better) |
| Backend | FastAPI + Uvicorn, local disk for jobs (no database) |
| Frontend | React + Vite + Tailwind, Three.js with GLTFLoader and OrbitControls |
| Tooling | Git with per-person branches, JSON schemas in `/shared/schemas` |

> **Tip:** This document stops your AI tool from silently switching approaches halfway through the build. If a tool suggests a different library, answer: "Not in Architecture.md. Keep the stack."
