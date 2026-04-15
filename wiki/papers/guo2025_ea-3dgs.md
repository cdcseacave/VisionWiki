---
title: "EA-3DGS: Efficient and Adaptive 3D Gaussians with Highly Enhanced Quality for Outdoor Scenes"
type: paper
tags: [3d-gaussian-splatting, outdoor-reconstruction, large-scale, mesh-guided-initialization, vector-quantization, pruning, densification]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/guo_2025_ea-3dgs.pdf
url: https://arxiv.org/abs/2505.10787
status: draft
---

📄 [Full paper](../../papers/radiance-fields/guo_2025_ea-3dgs.pdf) · [arXiv](https://arxiv.org/abs/2505.10787)

## TL;DR

EA-3DGS is a method for high-quality, efficient [[3d-gaussian-splatting]] of outdoor and large-scale scenes. It introduces an adaptive tetrahedral mesh for structured Gaussian initialization, contribution-aware pruning and structure-aware densification for efficient Gaussian management, and codebook-based vector quantization for 5.2x model compression with minimal quality loss. Evaluated on Mill 19, MatrixCity, WHU, Tanks & Temples, and self-collected aerial scenes.

## Problem

Standard [[3d-gaussian-splatting]] struggles with outdoor large-scale scenes because: (1) point-based initialization from sparse [[colmap|COLMAP]] points lacks coverage in low-texture regions, leading to poor geometry; (2) the default densification/pruning heuristics produce millions of redundant Gaussians, causing memory constraints; and (3) the resulting models are too large for storage and deployment.

## Method

1. **Adaptive tetrahedral mesh initialization**: From [[colmap|COLMAP]] sparse points, a Delaunay tetrahedralization is computed. Gaussian primitives are initialized on each triangular face of the tetrahedra, with adaptive mesh refinement based on local geometry. This provides structured coverage even in low-texture regions where sparse points are absent.
2. **Contribution-aware pruning**: A GS Score is computed for each Gaussian based on its actual contribution to rendered views (using ray-Gaussian intersection counts, opacity, and blending weight). Low-contribution Gaussians are pruned.
3. **Structure-aware densification**: Instead of densifying based purely on gradient magnitude, the method uses surface curvature (ratio of eigenvalues: rho = lambda_0 / (lambda_0 + lambda_1 + lambda_2)) to identify low-curvature regions that need more Gaussians, preserving geometric structure.
4. **Codebook vector quantization**: Gaussian parameters (except position) are quantized using a learned codebook, achieving approximately 5.2x compression (e.g., 1.8 GB to 345 MB) with only ~0.37 dB PSNR drop.

## Results

**Mill 19 dataset** (Rubble scene): PSNR 25.28 / SSIM 0.792 / LPIPS 0.337, outperforming 3DGS (25.16), Scaffold-GS (27.25 on Rubble but lower on Building), and NeRF-based methods. Best overall across both Mill 19 scenes.

**MatrixCity**: PSNR 21.58 / SSIM 0.720 / LPIPS 0.314, outperforming all compared methods on this synthetic urban dataset.

**Mill 19 Building scene**: PSNR 25.19 / SSIM 0.788 / LPIPS 0.287, best among all methods.

**Rendering speed**: ~48.89 FPS, comparable to vanilla 3DGS (~45.98 FPS) and far exceeding Mega-NeRF (0.008 FPS) and Switch-NeRF (0.007 FPS).

**Compression**: Quantized variant achieves PSNR 24.82 on Building at 345 MB vs. 1.8 GB uncompressed, a 5.2x reduction.

## Why it matters

EA-3DGS addresses the practical deployment challenges of Gaussian splatting for real-world outdoor and aerial reconstruction. The mesh-guided initialization is particularly significant for UAV photogrammetry where low-texture regions (roads, rooftops, terrain) are common. The compression via vector quantization makes large-scale models feasible for storage and streaming, bridging the gap between research quality and production deployment.

## Pipeline contribution

- **Adaptive tetrahedral-mesh initialization (N1)** — Delaunay tetrahedralization of COLMAP sparse points; Gaussians spawned on triangular faces with local-geometry refinement. candidate thread: [[radiance-field-evolution]] city-scale · stage: *Gaussian initialization* · replaces/augments: *COLMAP-sparse-point direct init* · expected gain: coverage in textureless regions (roads, rooftops, terrain) where sparse points are absent.
- **Contribution-aware pruning via GS Score (N2)** — per-Gaussian score from ray-intersection counts + opacity + blending weight; low-contribution pruned. candidate thread: [[radiance-field-evolution]] · stage: *densification control* · replaces/augments: *gradient-magnitude pruning* · expected gain: kills redundant Gaussians while preserving rendered contribution.
- **Structure-aware densification via curvature (N3)** — eigenvalue-ratio $\rho = \lambda_0/(\lambda_0+\lambda_1+\lambda_2)$ identifies low-curvature regions needing more Gaussians. candidate thread: [[radiance-field-evolution]] · stage: *densification target selection* · replaces/augments: *gradient-magnitude-only densification* · expected gain: preserves geometric structure under bulk densification.
- **Codebook vector quantization on non-position attributes (N4)** — learned codebook, 5.2× compression. candidate thread: [[radiance-field-evolution]] city-scale · stage: *storage/deployment* · replaces/augments: *LightGaussian / C3DGS approaches* · expected gain: 1.8 GB → 345 MB at 0.37 dB cost.
- **Role**: EA-3DGS is the **compression leg** of the three-way city-scale stack (partition [[lin2024_vastgaussian]] / incremental [[deng2026_vpgs-slam]] / compress EA-3DGS). Orthogonal to the other two; layering them is the open synthesis bet.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl et al., 2023) and addresses its limitations for outdoor scenes.
- Compared against [[Scaffold-GS]], Mega-NeRF, Switch-NeRF, GaMes, and Compressed-3DGS (C3DGS).
- Mesh initialization is related to GaMeS which also uses mesh structures but with a different approach.
- Compression via [[vector-quantization]] is related to LightGaussian and C3DGS but uses codebook quantization specifically.
- Uses [[colmap|COLMAP]] for initial sparse reconstruction and camera parameters.
- Evaluated on large-scale datasets: Mill 19, MatrixCity, WHU, Tanks & Temples, and self-collected SCUT-CA aerial scenes.

## Open questions / limitations

- The Delaunay tetrahedralization adds preprocessing time; scalability to very large point clouds (millions of points) may be a concern.
- The method is primarily validated on aerial/outdoor scenes; performance on indoor or object-scale scenes is less explored.
- The codebook quantization requires a separate fine-tuning phase; end-to-end quantization-aware training could be more efficient.
- Handling of dynamic elements in outdoor scenes (vehicles, pedestrians) is not addressed.
- The structure-aware densification heuristic based on curvature eigenvalues may need tuning for different scene types.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[colmap|COLMAP]]
- [[Scaffold-GS]]
- [[vector-quantization]]
- [[Delaunay-tetrahedralization]]
- [[large-scale-reconstruction]]
