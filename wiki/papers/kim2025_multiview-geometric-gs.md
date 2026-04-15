---
title: "Multiview Geometric Regularization of Gaussian Splatting for Accurate Radiance Fields"
type: paper
tags: [3d-gaussian-splatting, multi-view-stereo, surface-reconstruction, depth-regularization, normal-consistency, geometric-accuracy]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/kim_2025_multiview-geometric-gs.pdf
url: https://arxiv.org/abs/2506.13508
status: draft
---

📄 [Full paper](../../papers/radiance-fields/kim_2025_multiview-geometric-gs.pdf) · [arXiv](https://arxiv.org/abs/2506.13508)

## TL;DR

This paper proposes a multiview geometric regularization strategy for [[3d-gaussian-splatting]] that integrates [[multi-view-stereo]] (MVS) depth, RGB, and normal constraints into both initialization and optimization. The key insight is that MVS depth and GS-optimized depth are complementary: MVS is robust in high color variation regions (via patch matching and epipolar constraints), while GS provides better estimates near object boundaries. By combining both through a median depth-based relative depth loss with uncertainty estimation, the method achieves state-of-the-art surface reconstruction among explicit methods while maintaining high rendering quality.

## Problem

Methods like [[2d-gaussian-splatting|2D-Gaussian-Splatting]] and Gaussian Opacity Fields attempt to improve the geometric accuracy of [[3d-gaussian-splatting]], but still struggle to reconstruct smooth and reliable geometry, especially in scenes with significant color variation across viewpoints. This is due to per-point appearance modeling and single-view optimization constraints that lack multiview geometric consistency enforcement.

## Method

1. **MVS-guided initialization**: Dense depth maps from a multi-view stereo pipeline are used to generate initial Gaussian positions, avoiding suboptimal positions from sparse [[colmap|COLMAP]] points alone. This provides better coverage in textureless and high color-variation regions.
2. **Median depth-based relative depth loss**: Instead of using rendered mean depth (which is inaccurate for semi-transparent Gaussians), the method uses median depth. The relative depth loss compares depth ratios between neighboring pixels from MVS and rendered depths, with learned per-pixel uncertainty weights to downweight unreliable MVS estimates.
3. **Multiview extension**: The depth, normal consistency, and depth distortion losses are extended to virtual viewpoints neighboring the training views, allowing gradients to regularize more Gaussians per iteration.
4. **Multiview RGB loss**: Photometric consistency across views enforces appearance consistency.
5. **Normal and depth distortion losses**: Enforce surface smoothness and Gaussian compactness along rays.

## Results

**DTU dataset** (Chamfer distance, mm): Achieves state-of-the-art among explicit methods (comparable to Neuralangelo, the best implicit method). Training time is ~58 min + ~4 min for MVS depth estimation.

**Tanks and Temples** (F1 score): State-of-the-art among explicit methods. Training time ~75 min + ~35 min for MVS.

**Mip-NeRF 360** (rendering quality): Outperforms baselines in PSNR on outdoor scenes while maintaining competitive SSIM/LPIPS.

**Ablation**: Each component (multiview RGB loss, MVS-guided init, relative depth loss, multiview extension, normal/distortion losses) contributes incrementally. Median depth significantly outperforms mean depth for the relative depth loss.

## Why it matters

This work demonstrates that the complementary strengths of classical MVS and neural Gaussian splatting can be systematically combined. Rather than relying solely on photometric loss for geometry, incorporating MVS priors through a well-designed relative depth loss closes the gap between explicit and implicit methods for surface reconstruction, while retaining the fast rendering advantage of [[3d-gaussian-splatting]].

## Pipeline contribution

- **MVS-guided Gaussian initialization (N1)** — dense MVS depth replaces COLMAP sparse points as the init source. candidate thread: [[radiance-field-evolution]] · stage: *Gaussian initialization* · replaces/augments: *sparse-point init* · expected gain: coverage in textureless / high color-variation regions; input to every subsequent geometry regularization.
- **Median-depth-based relative depth loss with learned uncertainty (N2)** — median rather than mean depth; depth-ratio comparison between neighboring pixels; per-pixel uncertainty downweights unreliable MVS. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A · stage: *depth supervision loss* · replaces/augments: *absolute-depth L1 against MVS* · expected gain: robust to semi-transparent Gaussian ambiguity and MVS noise; the signature loss that establishes "external MVS > Gaussian self-supervision" on DTU/T&T.
- **Multiview extension of losses to virtual neighbor viewpoints (N3)** — regularizes more Gaussians per iteration via gradients from neighbor views. candidate thread: [[radiance-field-evolution]] · stage: *loss propagation* · expected gain: faster convergence on geometry.
- **Role**: Kim 2025 is the paper that formalized "external MVS > Gaussian self-supervision for mesh quality" — directly contradicted by [radl2026_confidence-mesh-3dgs] (CoMe), which argues self-supervised confidence wins. The unresolved contradiction is the open tension in [[gaussian-to-mesh-pipelines]] Current SOTA.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl et al., 2023) and extends it with geometric regularization.
- Improves upon [[2d-gaussian-splatting|2D-Gaussian-Splatting]] (Huang et al., 2024) and Gaussian Opacity Fields (GOF) (Yu et al., 2024) which also target geometric accuracy.
- Uses MVS depth priors, related to classical [[multi-view-stereo]] and [[colmap|COLMAP]] dense reconstruction.
- Compared against implicit methods: [[NeuS]], Geo-NeuS, [[Neuralangelo]].
- Compared against explicit methods: SuGaR, 3DGS, 2DGS, GOF.
- Evaluated on DTU, Tanks and Temples, and [[Mip-NeRF-360]] benchmarks.

## Open questions / limitations

- MVS depth estimation adds ~4-35 minutes overhead depending on scene scale, which may be a bottleneck for time-critical applications.
- The uncertainty estimation for MVS depth is learned per-pixel; a more principled geometric uncertainty model could improve robustness.
- Performance on highly reflective or transparent surfaces (where both MVS and GS struggle) is not extensively evaluated.
- The method is per-scene optimized; integration with feed-forward or generalizable Gaussian prediction is not explored.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[2d-gaussian-splatting|2D-Gaussian-Splatting]]
- [[multi-view-stereo]]
- [[colmap|COLMAP]]
- [[NeuS]]
- [[Neuralangelo]]
- [[Mip-NeRF-360]]
