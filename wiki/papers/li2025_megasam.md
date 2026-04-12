---
title: "MegaSaM: Accurate, Fast, and Robust Structure and Motion from Casual Dynamic Videos"
type: paper
tags: [sfm, slam, dynamic-scenes, monocular-depth, motion-segmentation, video-depth, differentiable-ba]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/li_2025_megasam.pdf
url: https://arxiv.org/abs/2412.04463
status: draft
---

📄 [Full paper](../../papers/sfm-slam/li_2025_megasam.pdf) · [arXiv](https://arxiv.org/abs/2412.04463)

## TL;DR

MegaSaM is a system for accurate, fast, and robust camera pose and depth estimation from casual monocular videos of dynamic scenes. It extends the [[DROID-SLAM]] differentiable BA framework with learned motion probability maps to handle dynamic content, monocular depth initialization, and uncertainty-aware global [[bundle-adjustment]] that adapts regularization based on the observability of scene geometry and camera parameters.

## Problem

Conventional SfM and SLAM assume predominantly static scenes with large parallax. Casual monocular videos often feature moving objects, limited camera motion (near-rotational), unknown focal length, and complex dynamics. Existing methods either fail outright on such videos, require expensive test-time optimization (CasualSAM, RoDynRF), or are brittle (MonST3R). The core challenge is disentangling camera ego-motion from object motion in a differentiable optimization framework.

## Method

MegaSaM builds on [[DROID-SLAM]]'s differentiable BA layer with three key innovations:

1. **Learned Motion Probability Maps**: A separate network $F_m$ predicts per-pixel object movement probability, trained on dynamic synthetic data in a two-stage scheme: first ego-motion pretraining on static scenes (learning flow and confidence), then dynamic finetuning with frozen flow model (learning movement maps via cross-entropy loss). This decoupling is critical for stable training of the differentiable BA.

2. **Monocular Depth Initialization**: Per-frame disparity is initialized using aligned monocular depth from DepthAnything + UniDepth (for metric scale/shift and focal length estimates), replacing DROID-SLAM's constant initialization. Camera poses are initialized via depth-only BA before joint optimization.

3. **Uncertainty-Aware Global BA**: Uses the diagonal of the approximate Hessian to estimate epistemic uncertainty of disparity and focal length parameters. Mono-depth regularization weight is set adaptively: $w_d = \gamma_d \exp(-\beta_d \cdot \text{med}(\text{diag}(H_d)))$. Focal length optimization is disabled when its Hessian entry falls below a threshold, indicating unobservability.

An optional consistent depth optimization module refines video depths at full resolution using optical flow reprojection, temporal consistency, and surface normal losses, without expensive network finetuning.

## Results

- **Sintel**: ATE 0.018 (calibrated), 0.023 (uncalibrated) vs. next best CasualSAM 0.036/0.067. Depth abs-rel 0.21 vs. MonST3R 0.31.
- **DyCheck**: ATE 0.020 (calibrated), 0.020 (uncalibrated) vs. ACE-Zero 0.062/0.056. Depth abs-rel 0.11 vs. MonST3R 0.26.
- **In-the-Wild videos**: ATE 0.004 (calibrated/uncalibrated) vs. CasualSAM 0.031/0.035.
- **Runtime**: ~1 second per frame, competitive with MonST3R and significantly faster than CasualSAM (1-3 minutes) and RoDynRF (6-15 minutes).
- Trained on synthetic data only (TartanAir, Kubric) but generalizes strongly to real-world videos.

## Why it matters

MegaSaM demonstrates that a carefully modified deep visual SLAM framework can handle the full complexity of in-the-wild dynamic videos without requiring scene-specific optimization or radiance field fitting. The uncertainty-aware BA provides principled handling of degenerate camera motions (rotation-dominant, forward-only) that commonly occur in casual capture, making SfM accessible to non-expert users.

## Relation to prior work

- Directly extends [[DROID-SLAM]]'s differentiable BA framework with dynamic scene handling.
- Competes with CasualSAM (test-time depth network finetuning), MonST3R (extends [[DUSt3R]] to dynamic scenes), Particle-SfM, and LEAP-VO.
- Uses [[DepthAnything]] and UniDepth for monocular depth initialization; contrasts with MonST3R's approach of predicting 3D pointmaps from [[DUSt3R]].
- Consistent depth module follows CasualSAM's formulation but avoids expensive network finetuning.
- Motion probability maps are conceptually related to Particle-SfM's motion segmentation but learned end-to-end within the BA framework.

## Open questions / limitations

- Can fail when moving objects dominate the entire image or there is nothing static to track reliably.
- Trained only on synthetic data; real-world dynamic training data could improve performance further.
- Low-resolution disparity maps (1/8 resolution) from the SLAM module may miss fine geometric details.
- Assumes a single shared focal length across the video.

## References added to the wiki

- [[DROID-SLAM]]
- [[DUSt3R]]
- [[bundle-adjustment]]
- [[monocular-depth-estimation]]
- [[DepthAnything]]
- [[structure-from-motion]]
- [[motion-segmentation]]
- [[optical-flow]]
- [[Levenberg-Marquardt]]
- [[differentiable-rendering]]
