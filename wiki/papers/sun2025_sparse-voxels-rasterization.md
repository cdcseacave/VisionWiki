---
title: "Sparse Voxels Rasterization: Real-time High-fidelity Radiance Field Rendering"
type: paper
tags: [voxel, radiance-field, rasterization, novel-view-synthesis, real-time-rendering, neural-free]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/sun_2025_sparse-voxels-rasterization.pdf
url: https://arxiv.org/abs/2412.04459
status: draft
---

📄 [Full paper](../../papers/radiance-fields/sun_2025_sparse-voxels-rasterization.pdf) · [arXiv](https://arxiv.org/abs/2412.04459)

## TL;DR

SVRaster proposes an efficient radiance field rendering framework that rasterizes adaptive sparse voxels instead of using neural networks or 3D Gaussians. By leveraging ray direction-dependent Morton ordering for correct depth sorting and adaptive multi-resolution voxel allocation (up to 65536^3 grid resolution), it achieves state-of-the-art novel-view synthesis quality comparable to [[Zip-NeRF]] and [[3d-gaussian-splatting]] while maintaining real-time rendering speed, and avoids the popping artifacts inherent to Gaussian splatting.

## Problem

[[3d-gaussian-splatting]] has two fundamental limitations: (1) sorting Gaussians by center position does not guarantee correct depth ordering, causing popping artifacts when views change, and (2) the volume density at a 3D point covered by multiple Gaussians is ill-defined, making surface reconstruction non-trivial. Prior neural-free voxel methods achieve correct ordering and well-defined volumes but rely on slow ray casting, making them impractical for real-time applications.

## Method

1. **Adaptive sparse voxel representation**: Voxels are allocated at multiple levels of detail within an octree structure, achieving up to 65536^3 effective resolution. Each voxel stores density and color parameters (using [[spherical-harmonics]] or learned features) without neural networks.
2. **Sparse voxel rasterizer**: A custom CUDA rasterizer sorts voxels using ray direction-dependent Morton codes (16 levels, 3 bits each = 48-bit keys) to ensure correct depth ordering for each pixel. This combines the efficiency of tile-based rasterization (as in 3DGS) with the correct ordering guarantees of volume rendering.
3. **Post-activation density**: Voxel density uses a post-activation formulation that can model arbitrarily sharp boundaries, unlike Gaussian falloff. The average number of primitives per pixel is 27 (vs. 63 for 3DGS), improving efficiency.
4. **Compatibility**: The voxel representation is directly compatible with TSDF Fusion, Marching Cubes, Voxel Pooling, and vision foundation model feature lifting (RADIO, Segformer).

## Results

**Mip-NeRF 360 dataset**: Achieves significantly better LPIPS than [[3d-gaussian-splatting]] and surpasses [[Zip-NeRF]] on perceptual quality. Rendering speed is comparable to 3DGS on average (scene-dependent). Improves over the previous best neural-free voxel model by over 4 dB PSNR and more than 10x FPS speedup.

- Recovers finer details than 3DGS in qualitative comparisons.
- No popping artifacts due to correct depth ordering.
- Fast-rendering variant achieves >2x FPS with only moderate quality reduction.
- Fast-training variant provides >3x training speedup.
- Seamless surface extraction via Marching Cubes from the voxel density field.

## Why it matters

SVRaster demonstrates that voxels -- a classical 3D representation -- can match or exceed the quality of Gaussian splatting when combined with modern rasterization techniques. By eliminating the need for neural networks and 3D Gaussians while providing correct depth ordering and well-defined volume density, it opens the door to a new class of efficient, artifact-free radiance field methods that are natively compatible with decades of grid-based 3D processing tools.

## Pipeline contribution

- **Ray-direction Morton-ordered sparse-voxel rasterizer (N1)** — 48-bit Morton codes sort voxels correctly per pixel, combining tile-based speed with volume-rendering correctness. candidate thread: [[radiance-field-evolution]] neural-free lane · stage: *rasterization primitive* · replaces/augments: *3DGS depth-sort by center (incorrect) / ray-casting (slow)* · expected gain: no popping artifacts; comparable FPS to 3DGS with sharper details.
- **Post-activation density (N2)** — sharp boundaries modelable, unlike Gaussian falloff. candidate thread: [[radiance-field-evolution]] · stage: *density parameterization* · expected gain: 27 primitives/pixel (vs 63 for 3DGS); finer detail recovery.
- **Native compatibility with TSDF / marching cubes / voxel pooling / VFM feature lifting (N3)** — voxel representation plugs into 30 years of grid-based tooling. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm C · stage: *primitive with direct mesh extraction* · expected gain: no TSDF fusion post-pass needed.
- **Role**: SVRaster is the *alternative-primitive* lane in [[radiance-field-evolution]]. Every synthesis bet of the form "3DGS with X regularization" has a natural counterpart "SVRaster with X" that hasn't been tried.

## Relation to prior work

- Positions as an alternative to [[3d-gaussian-splatting]] (Kerbl et al., 2023) with better geometric properties.
- Builds on voxel-based radiance fields like Plenoxels and DVGO, but replaces ray casting with a custom rasterizer for speed.
- Compared against [[Zip-NeRF]], [[Mip-NeRF-360]], [[3d-gaussian-splatting]], Mip-Splatting, and Scaffold-GS.
- Compatible with [[TSDF-fusion]] and [[Marching-Cubes]] for surface extraction.
- Uses [[spherical-harmonics]] for view-dependent color, similar to 3DGS.

## Open questions / limitations

- Rendering FPS varies significantly across scenes due to differences in scene complexity and voxel density distribution; not always faster than 3DGS.
- Sorting 48-bit Morton codes is more expensive than 3DGS's 32-bit float sorting.
- The adaptive voxel allocation requires careful hyperparameter tuning for different scene types.
- Memory consumption for very high-resolution grids (65536^3) may be a concern for resource-constrained devices.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[Zip-NeRF]]
- [[Mip-NeRF-360]]
- [[spherical-harmonics]]
- [[TSDF-fusion]]
- [[Marching-Cubes]]
- [[volume-rendering]]
