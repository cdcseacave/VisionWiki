---
title: "GeoSVR: Taming Sparse Voxels for Geometrically Accurate Surface Reconstruction"
type: paper
tags: [sparse-voxels, surface-reconstruction, radiance-fields, depth-estimation, multi-view]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/li_2025_geosvr.pdf
url: https://arxiv.org/abs/2509.18090
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/li_2025_geosvr.pdf) · [arXiv](https://arxiv.org/abs/2509.18090)

## TL;DR

GeoSVR is an explicit sparse-voxel-based framework for surface reconstruction from multi-view images that overcomes the representational bottlenecks of Gaussian Splatting methods. It introduces voxel-uncertainty depth constraints and sparse voxel surface regularization to achieve state-of-the-art geometric accuracy on DTU, Tanks and Temples, and Mip-NeRF 360, while maintaining training efficiency competitive with 3DGS-based methods.

## Problem

Prevailing surface reconstruction approaches based on [[3d-gaussian-splatting]] are increasingly constrained by representational bottlenecks, leading to rough, inaccurate, or incomplete surface recovery. Methods relying on external depth estimators may degrade quality if the cues are noisy. The potential of explicit sparse voxels for accurate geometry learning had not been fully explored, especially for detailed surface reconstruction with high completeness.

## Method

GeoSVR builds on the SVRaster sparse voxel rasterization framework and introduces three key components:

1. **Voxel-Uncertainty Depth Constraint**: Evaluates per-voxel uncertainty to maximally utilize external depth cues (e.g., from monocular depth estimators) while avoiding quality degradation from noisy priors. Only voxels with low uncertainty receive depth supervision.

2. **Sparse Voxel Surface Regularization**: Enforces global geometry consistency across voxels by regularizing the voxel-based surface formation. This encourages sharp, accurate surfaces and removes artifacts/redundant voxels.

3. **Multi-View Regularization**: Enlarges the global geometry consistency constraint across views for tiny voxels, facilitating complete and detailed reconstruction.

Surface extraction uses the opacity field of the sparse voxels, enabling dense mesh extraction via [[marching-cubes]].

## Results

- **DTU**: Achieves the best overall Chamfer distance among all baselines, surpassing both SDF-based methods (Geo-NeuS) and 3DGS-based methods (PGSR), including those using external geometry cues.
- **Tanks and Temples**: Best F1-score, outperforming Neuralangelo, MonoGSDF, and PGSR.
- **Mip-NeRF 360**: Competitive novel view synthesis (PSNR/SSIM/LPIPS) among surface reconstruction methods.
- **Efficiency**: Training time competitive with 3DGS-based methods; inference at 83--144 FPS depending on dataset. Mesh extraction within minutes.

## Why it matters

GeoSVR demonstrates that explicit sparse voxels -- rather than Gaussians or implicit neural fields -- can achieve top-tier surface reconstruction quality with high efficiency. It revives interest in voxel-based representations as a viable alternative to the dominant [[3d-gaussian-splatting]] paradigm, particularly for geometry-focused tasks where Gaussians' unstructured nature is a limitation.

## Pipeline contribution

- **Voxel-uncertainty depth constraint (N1)** — per-voxel uncertainty gates external depth supervision, accepting noisy priors only where confidence is high. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm C · stage: *external depth fusion* · replaces/augments: *uniform MVS depth supervision* · expected gain: robust to noisy priors; SOTA Chamfer on DTU.
- **Sparse voxel surface regularization (N2)** — global geometry consistency across voxels encourages sharp surfaces and removes redundant voxels. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm C · stage: *voxel-field regularization* · replaces/augments: *3DGS surface-flattening regularizers (PGSR, VA-GS)* · expected gain: explicit-voxel counterpart to 3DGS surface regularization, cleaner because voxels have well-defined volume.
- **Multi-view regularization for tiny voxels (N3)** — extends consistency across views to preserve small structures. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *multi-view loss on voxels* · expected gain: completeness in fine-detail regions.
- **Role**: GeoSVR is the **Paradigm-C exemplar** — natively mesh-extractable, Gaussian-free, SOTA DTU. The synthesis bet *GeoSVR + CoMe + MVS supervision* flagged in the thread combines this paper's N1 with the confidence-weighting from [radl2026_confidence-mesh-3dgs].

## Relation to prior work

- Builds on **SVRaster** for efficient sparse voxel rasterization.
- Competes with and outperforms [[3d-gaussian-splatting]]-based surface methods: [[2d-gaussian-splatting]], GOF, PGSR, GS2Mesh, VCR-GauS, MonoGSDF.
- Compares favorably to [[neural-implicit-surfaces]] / [[signed-distance-field|SDF]]-based approaches: NeuS, Geo-NeuS, Neuralangelo, VolSDF.
- Contrasts with methods requiring external normal estimators or heavy MLP-based implicit fields.

## Open questions / limitations

- Multi-view regularization increases training time due to less efficient code implementation (acknowledged by authors as future work).
- Potential for further quality improvement by combining with learned geometric priors.
- Scalability to very large-scale unbounded outdoor scenes not fully explored.

## References added to the wiki

- [[li2025_geosvr]] (this page)
