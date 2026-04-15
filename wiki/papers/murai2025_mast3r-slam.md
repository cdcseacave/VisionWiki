---
title: "MASt3R-SLAM: Real-Time Dense SLAM with 3D Reconstruction Priors"
type: paper
tags: [slam, mast3r, dense-slam, monocular, uncalibrated, real-time, pointmaps]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/sfm-slam/murai_2025_mast3r-slam.pdf
url: https://arxiv.org/abs/2412.12392
status: draft
---

📄 [Full paper](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf) · [arXiv](https://arxiv.org/abs/2412.12392)

## TL;DR

MASt3R-SLAM is the first real-time dense monocular SLAM system built on two-view 3D reconstruction priors from [[mast3r|MASt3R]]. It operates at 15 FPS, handles generic time-varying camera models (no known calibration required), and produces globally consistent poses and dense geometry through efficient pointmap matching, local fusion, loop closure, and second-order Sim(3) global optimization.

## Problem

Dense monocular SLAM that provides both accurate poses and consistent dense maps without known camera calibration does not exist as a plug-and-play solution. Existing approaches either require depth sensors, known intrinsics, or slow camera motion. Single-view priors (monocular depth) are ambiguous and inconsistent across views. Multi-view priors (optical flow) entangle pose and geometry. Two-view 3D priors like [[dust3r|DUSt3R]]/[[mast3r|MASt3R]] produce direct 3D measurements but their offline SfM pipelines (MASt3R-SfM) do not scale to real-time and have quadratic complexity.

## Method

The system introduces efficient methods for each SLAM component:

1. **Pointmap Matching**: Instead of expensive feature matching or k-d tree search, iteratively projects 3D points from one pointmap into the other using the generic camera model defined by normalized rays. Converges within 10 iterations per pixel, runs in 2ms via custom CUDA kernels. Optional feature refinement with MASt3R descriptors in a local patch window.

2. **Tracking and Pointmap Fusion**: Estimates relative Sim(3) pose by minimizing a directional ray error (more robust than 3D point error against depth prediction errors). Fuses pointmaps via running weighted average filter, effectively averaging out noise and building a consistent camera model over time.

3. **Graph Construction and Loop Closure**: Incremental ASMK-based image retrieval using MASt3R encoded features. Loop closure candidates are verified through MASt3R decoder and match count thresholds.

4. **Second-Order Global Optimization**: Gauss-Newton optimization over Sim(3) poses minimizing ray errors across all graph edges, with sparse Cholesky decomposition. Analytical Jacobians in CUDA. Converges within 10 iterations per new keyframe.

The system's only assumption is a generic central camera model (unique camera centre per frame).

## Results

- **TUM RGB-D**: Average ATE 0.030m (calibrated), best among all methods including DROID-SLAM (0.038m). Uncalibrated: 0.060m, significantly better than DROID-SLAM* with GeoCalib (0.158m).
- **7-Scenes**: ATE 0.047m (calibrated), outperforming DROID-SLAM (0.049m) and NICER-SLAM (0.086m). Chamfer distance 0.066m, better than DROID-SLAM (0.077m).
- **ETH3D-SLAM**: Best mean ATE (0.086m) and AUC (23.935) among monocular methods, demonstrating superior robustness on this challenging dataset.
- **EuRoC**: ATE 0.041m, competitive with DROID-SLAM (0.022m), with better Chamfer distance (0.085m vs 0.117m).
- **Runtime**: 15 FPS with matching at 2ms (40x faster than MASt3R's native matching at 2 seconds).

## Why it matters

MASt3R-SLAM demonstrates that off-the-shelf two-view 3D reconstruction priors can serve as a universal foundation for real-time dense SLAM, challenging the DROID-SLAM paradigm that has dominated recent SLAM research. The ability to operate without known camera calibration opens SLAM to truly in-the-wild applications with arbitrary camera hardware.

## Pipeline contribution

- **Iterative pointmap projection matching via generic-camera rays (N1)** — converges 10 iterations/pixel in 2ms via custom CUDA, replaces feature matching / k-d tree search. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *frame-to-frame matching on pointmaps* · replaces/augments: *MASt3R decoder match @ 2s/pair* · expected gain: 1000× matching speedup; enables real-time SLAM.
- **Directional ray-error pose estimation (N2)** — Sim(3) pose minimizing ray-direction residual instead of 3D point residual. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *pose estimation from pointmap pairs* · replaces/augments: *3D-point-distance minimization* · expected gain: robust to MASt3R depth-prediction errors (scale / bias).
- **Running weighted-average pointmap fusion (N3)** — online filter over MASt3R predictions per keyframe. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *temporal geometry fusion* · replaces/augments: *single-frame MASt3R output* · expected gain: noise averaging, consistent camera model over time.
- **Gauss-Newton Sim(3) global optimization with analytic CUDA Jacobians (N4)** — sparse Cholesky over pose graph. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *global optimization* · replaces/augments: *DROID-SLAM's differentiable BA* · expected gain: faster, calibration-free; 15 FPS end-to-end.
- **Role**: first real-time feed-forward SLAM on DUSt3R/MASt3R priors; establishes that two-view 3D priors + ray-based optimization can replace DROID-SLAM.

## Relation to prior work

- Built entirely on [[mast3r|MASt3R]] (successor of [[dust3r|DUSt3R]]) as the geometric prior, contrasting with DROID-SLAM's end-to-end learned approach.
- Uses [[mast3r|MASt3R]]-SfM's ASMK retrieval framework, adapted for incremental online operation.
- Ray-based error formulation inspired by generic camera calibration methods, avoiding parametric camera model assumptions.
- Competes with DROID-SLAM, DPV-SLAM, GO-SLAM, and NICER-SLAM.
- Pointmap fusion is inspired by classical filtering approaches in SLAM (e.g., EKF-SLAM), applied to MASt3R predictions.

## Open questions / limitations

- Does not refine all geometry in global optimization (only poses); a method for making pointmaps globally consistent in 3D while retaining coherence would be valuable.
- MASt3R is only trained on pinhole images; geometry predictions degrade with increasing distortion.
- Full-resolution decoder is a bottleneck for low-latency tracking and loop closure candidate checking.
- Scale inconsistency across MASt3R predictions requires Sim(3) optimization rather than SE(3).

## References added to the wiki

- [[mast3r|MASt3R]]
- [[dust3r|DUSt3R]]
- [[SLAM]]
- [[droid-slam|DROID-SLAM]]
- [[Sim3]]
- [[bundle-adjustment]]
- [[loop-closure]]
- [[ASMK]]
- [[Gauss-Newton]]
- [[generic-camera-model]]
