---
title: "DroneSplat: 3D Gaussian Splatting for Robust 3D Reconstruction from In-the-Wild Drone Imagery"
type: paper
tags: [3d-gaussian-splatting, drone, aerial-reconstruction, dynamic-distractors, sparse-view, multi-view-stereo]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/tang_2025_dronesplat.pdf
url: https://arxiv.org/abs/2503.16964
status: draft
---

📄 [Full paper](../../papers/radiance-fields/tang_2025_dronesplat.pdf) · [arXiv](https://arxiv.org/abs/2503.16964)

## TL;DR

DroneSplat is a [[3d-gaussian-splatting]] framework designed for robust 3D reconstruction from in-the-wild drone imagery. It addresses two core challenges: dynamic distractors violating the static-scene assumption and viewpoint sparsity from limited drone coverage. An adaptive local-global masking strategy handles dynamics, while [[multi-view-stereo]] predictions and a voxel-guided optimization strategy improve geometry under sparse views.

## Problem

Drone-captured imagery in wild environments contains moving objects (vehicles, people) that violate the static-scene assumption of [[nerf|NeRF]] and [[3d-gaussian-splatting]], causing artifacts. Additionally, individual scene regions may only be covered by a few drone views, leading to overfitting and poor novel-view quality. Existing methods either rely on predefined semantic categories for distractor removal or apply hard thresholds that do not adapt across scenes and training stages.

## Method

1. **Adaptive Local-Global Masking**: Combines residual-based statistical detection with pixel-level segmentation. The local masking threshold adapts based on real-time reconstruction residuals and segmentation results. Global masking tracks high-residual candidates across the full scene context. This avoids fixed thresholds and predefined distractor categories.
2. **Geometric-Aware Point Sampling**: Uses a multi-view stereo model (DUSt3R-based) to predict dense 3D points, providing rich geometric priors for Gaussian initialization beyond [[colmap|COLMAP]] sparse points.
3. **Voxel-Guided Optimization**: Constructs a voxel grid from MVS predictions and applies depth distortion loss and geometric constraints to regularize [[3d-gaussian-splatting]] optimization, preventing overfitting in sparse-view regions.
4. **DroneSplat Dataset**: A new benchmark of 24 drone-captured sequences with dynamic and static scenes at varying dynamics levels.

## Results

**Distractor elimination** (DroneSplat dynamic dataset): Outperforms all baselines (3DGS, GS-W, WildGaussians, RobustNeRF, NeRF On-the-go, NeRF-HuGS, Mip-Splatting) on PSNR/SSIM/LPIPS across low, medium, and high dynamic scenes. Also achieves best results on the NeRF On-the-go dataset.

**Limited-view reconstruction** (DroneSplat static + UrbanScene3D, 6 input views): Outperforms FSGS and Scaffold-GS sparse-view methods. On Simingshan scene: PSNR 22.46 / SSIM 0.728 / LPIPS 0.156 vs. next-best PSNR 20.41.

**Joint challenge** (dynamic + sparse): On Simingshan with 6 views and distractors, achieves PSNR 22.46 vs. next-best 20.41.

## Why it matters

DroneSplat is among the first methods to jointly handle dynamic distractors and viewpoint sparsity for drone-based reconstruction, which is the realistic operating condition for aerial surveying. The adaptive masking removes the need for scene-specific threshold tuning, and the MVS-guided optimization demonstrates that combining feed-forward stereo priors with per-scene [[3d-gaussian-splatting]] optimization is a powerful paradigm.

## Pipeline contribution

- **Adaptive local-global distractor masking (N1)** — residual-based statistical detection + pixel-level segmentation, with thresholds adapting via real-time residuals. candidate thread: [[radiance-field-evolution]] aerial/in-the-wild · stage: *distractor-aware photometric supervision* · replaces/augments: *fixed-threshold / semantic-category masks (RobustNeRF, NeRF-HuGS)* · expected gain: handles all dynamic-distractor levels (low/med/high) without per-scene tuning; best PSNR/SSIM/LPIPS on DroneSplat + NeRF On-the-go datasets.
- **Geometric-aware point sampling from DUSt3R-based MVS (N2)** — dense 3D points from a feed-forward stereo model as initialization. candidate thread: [[radiance-field-evolution]] · stage: *Gaussian initialization* · replaces/augments: *COLMAP sparse points* · expected gain: robust init on sparse drone views; supersedes earlier "InstantSplat" style init with DUSt3R-specific voxel-guidance.
- **Voxel-guided optimization with depth-distortion + geometric constraints (N3)** — voxel grid from MVS regularizes 3DGS training under sparse views. candidate thread: [[radiance-field-evolution]] · stage: *sparse-view regularization* · expected gain: +2 dB over next-best sparse-view baseline on 6-view Simingshan.
- **Role**: DroneSplat is the **aerial + in-the-wild** exemplar in [[radiance-field-evolution]]; the adaptive-masking mechanism is the candidate component that VastGaussian's pipeline currently lacks (VastGaussian has appearance drift, DroneSplat has dynamic distractors — they target different failure modes and should compose cleanly).

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] (Kerbl et al., 2023) as the core representation.
- Extends distractor handling ideas from RobustNeRF and NeRF On-the-go to the Gaussian splatting setting with adaptive thresholds.
- Uses [[multi-view-stereo]] predictions (DUSt3R) for initialization, similar to InstantSplat but with added voxel-guided optimization.
- Compared against [[Mip-Splatting]], GS-W, WildGaussians, FSGS, and [[Scaffold-GS]].
- Related to [[colmap|COLMAP]]-free reconstruction paradigms.

## Open questions / limitations

- The DroneSplat dataset, while novel, is relatively small (24 sequences). Generalization to more diverse environments needs further validation.
- The MVS model (DUSt3R) adds computational overhead for dense point prediction.
- Performance on scenes with very large dynamic objects (e.g., construction equipment occupying most of the frame) is not extensively evaluated.
- The framework currently treats all non-static elements as distractors; modeling dynamic objects explicitly could be valuable.

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[multi-view-stereo]]
- [[nerf|NeRF]]
- [[colmap|COLMAP]]
- [[Mip-Splatting]]
- [[Scaffold-GS]]
