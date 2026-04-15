---
title: "SOF: Sorted Opacity Fields for Fast Unbounded Surface Reconstruction"
type: paper
tags: [gaussian-splatting, opacity-fields, mesh-extraction, unbounded-scenes, surface-reconstruction]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/radl_2025_sof.pdf
url: https://arxiv.org/abs/2506.19139
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/radl_2025_sof.pdf) · [arXiv](https://arxiv.org/abs/2506.19139)

## TL;DR

SOF (Sorted Opacity Fields) introduces hierarchical resorting and a robust opacity field formulation to extract high-quality unbounded meshes from [[3d-gaussian-splatting]] representations, accelerating optimization by over 3x and meshing by up to 10x compared to Gaussian Opacity Fields (GOF), while producing more detailed meshes with fewer artifacts.

## Problem

Extracting accurate surfaces from 3D Gaussian representations in large-scale unbounded environments remains difficult. Existing methods like GOF rely on approximate depth estimates and global sorting heuristics, which introduce artifacts and limit mesh fidelity. The meshing step itself is also very slow (30+ minutes for GOF vs. ~3--5 minutes for SOF).

## Method

SOF improves upon the Gaussian Opacity Fields framework with:

1. **Hierarchical Resorting**: Instead of using a single global sorting order for all Gaussians, SOF introduces a hierarchical sorting strategy that provides more accurate per-ray ordering, reducing artifacts caused by incorrect primitive ordering.

2. **Robust Opacity Field Formulation**: A reformulated opacity field that better handles the transition from volumetric Gaussian representations to surface-like opacity, enabling more precise level-set extraction via [[marching-cubes]].

3. **Efficient Meshing Pipeline**: The combination of improved sorting and opacity formulation enables significantly faster mesh extraction from the opacity field using [[marching-cubes]] on a voxel grid.

The method works with standard 3DGS training and extracts meshes from the learned opacity field.

## Results

- **DTU**: Average Chamfer distance of 0.47 (competitive with GOF at 0.46), with training in ~16.7 min vs. >1 hour for GOF.
- **Tanks and Temples**: F1-score of 0.535 (Barn) and 0.736 (Truck) vs. GOF's 0.484 and 0.674, with 3x faster optimization and up to 10x faster meshing.
- **Mip-NeRF 360**: Competitive novel view synthesis metrics (PSNR/SSIM/LPIPS) with slightly reduced image quality compared to GOF.
- Overall: 3x faster training, up to 10x faster mesh extraction, with equal or better mesh quality.

## Why it matters

SOF makes high-quality mesh extraction from Gaussian Splatting practical for large-scale scenes by dramatically reducing computation time. The hierarchical resorting insight addresses a fundamental limitation of opacity-field-based mesh extraction: that global sorting heuristics degrade surface quality. This work is a direct stepping stone to the authors' later work on confidence-based mesh extraction.

## Pipeline contribution

- **Hierarchical per-ray resorting (N1)** — replaces GOF's global sort with per-ray hierarchical order. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A · stage: *opacity-field construction* · replaces/augments: *GOF global sort heuristic* · expected gain: correct primitive ordering, fewer meshing artifacts.
- **Robust opacity-field formulation (N2)** — reformulation that handles volumetric-to-surface transition for precise level-set extraction. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A · stage: *opacity → surface conversion* · replaces/augments: *GOF's opacity field* · expected gain: cleaner marching-cubes extraction.
- **Efficient meshing pipeline (N3)** — consequence of N1+N2. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *mesh extraction runtime* · expected gain: 3× faster training, up to 10× faster mesh extraction (30 min → 3–5 min).
- **Role**: SOF is the **meshing-pipeline backend** used by [radl2026_confidence-mesh-3dgs] (CoMe). CoMe's contributions sit on top; SOF provides the fast level-set extraction.

## Relation to prior work

- Directly extends **Gaussian Opacity Fields (GOF)** with improved sorting and opacity formulation.
- Builds on [[3d-gaussian-splatting]] and its surface-oriented variants: [[2d-gaussian-splatting]], PGSR, RaDe-GS.
- Mesh extraction via [[marching-cubes]] on the opacity/level-set field.
- Contrasts with [[signed-distance-field|SDF]]-based methods (NeuS, VolSDF, Neuralangelo) which are slower but may provide smoother surfaces.
- Compared against 3DGS, Mip-Splatting, GOF, [[2d-gaussian-splatting]] for NVS.

## Open questions / limitations

- Slightly reduced image quality compared to GOF in novel view synthesis (tradeoff for speed).
- Unbounded scene handling still depends on background model / contraction strategies.
- Opacity field formulation may still struggle with semi-transparent objects.

## References added to the wiki

- [[radl2025_sof]] (this page)
