# PRD.md: Product Requirement Document

*The foundation. Nothing gets built before this exists.*

## What to Build

**FloorForge** is a web app that turns a 2D floor-plan image (scan, photo, CAD export or architect drawing) into a **correctly scaled, navigable 3D model**. A trained segmentation model finds walls, doors, windows and rooms. Geometry code cleans them into real wall segments. Printed dimension labels are read by OCR to recover true metric scale. A validator checks the layout, and a 3D builder exports a GLB shown in a browser viewer with a room-dimensions table, a confidence overlay and a download button. It targets HackNex 2026 PS06, Mode A (floor plan → 3D).

## Target Users

- **Primary:** interior designers, junior architects, real-estate agents and students who have a 2D plan and need a quick 3D walkthrough. Low technical skill; they upload an image and expect a result in seconds.
- **Secondary (for the hackathon):** judges who bring unseen plans with measured dimensions and compare results against a baseline.
- **Problem:** modelling a plan in 3D by hand takes hours, and existing quick tools often get walls and dimensions wrong without saying so.

## Features of the App

### Must-have

- Upload a plan image (PNG/JPG), with 3 built-in sample plans.
- U-Net segmentation of wall / door / window / room, trained on CubiCasa5K (verify license).
- Vectorization: clean wall segments, snapped to right angles and shared endpoints; doors and windows attached to walls.
- Scale recovery: OCR of dimension labels + robust solver; door-width fallback labelled "estimated"; manual scale override.
- Topology validator: closed rooms, openings lie on walls, plausible sizes; warnings are shown, never hidden.
- 3D builder: walls with thickness, door and window openings, floors; export GLB.
- Viewer: orbit/zoom, plan overlay toggle, room dimensions table, scale-method badge, GLB download.
- Evaluation script that outputs layout IoU and dimension error, plus the ablation table versus the baseline.
- Short write-up (required by the PS).

### Nice-to-have (cut first if time runs out, in this order)

1. Confidence colouring in the overlay and 3D view.
2. Measure tool inside the 3D viewer.
3. Room labels read from plan text (Bedroom, Kitchen) shown in 3D.
4. Door-swing arcs detected and animated.
5. Synthetic plan-style augmentation generator.
6. Extra export formats (USD/IFC).

## Out of Scope

Room-video → 3D (Mode B), furniture reconstruction, multi-floor buildings, real-time processing, user accounts.

## Success Criteria (map to the judging rubric, Mode A)

| Rubric item (weight) | What we do about it | How we measure |
|---|---|---|
| Layout accuracy vs baseline (40%) | U-Net + line snapping + OCR scale solver | Layout IoU, dimension error % on own held-out set |
| Completeness (15%) | Topology validator and opening attachment | Rooms/openings found vs ground truth |
| Model quality (15%) | Correct thickness, heights, sills, scale | Visual check + measured wall lengths |
| Usability in viewer (10%) | Standard GLB, Three.js viewer, download | Opens in viewer and in a second 3D tool |
| Research contribution (20%) | Ablation: each added step vs baseline | Auto-generated ablation table |

## Constraints and Assumptions

- 24 hours, 3 people, free Colab T4 GPU (or a local GPU), free libraries only.
- Judges bring unseen plans with measured dimensions; styles may differ from CubiCasa5K, so robustness is a priority.
- The baseline is chosen by us (for example a classical OpenCV parser, plus an open-source floor-plan parser if one installs cleanly). We report results honestly, including where we lose.
