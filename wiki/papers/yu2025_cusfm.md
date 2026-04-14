---
title: "CuSfM: CUDA-Accelerated Structure-from-Motion"
type: paper
tags: [sfm, cuda, gpu-acceleration, autonomous-driving, extrinsic-refinement, nvidia]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/yu_2025_cusfm.pdf
url: https://arxiv.org/abs/2510.15271
status: draft
---

📄 [Full paper](../../papers/sfm-slam/yu_2025_cusfm.pdf) · [arXiv](https://arxiv.org/abs/2510.15271)

## TL;DR

CuSfM is a CUDA-accelerated offline [[structure-from-motion]] system from NVIDIA designed for autonomous driving scenarios. It leverages GPU parallelization with computationally intensive yet accurate feature extractors (ALIKED via TensorRT), non-redundant data association, and a multi-stage factor graph optimization to achieve significantly improved accuracy and speed over [[COLMAP]] and [[GLOMAP]], with additional support for multi-camera rig extrinsic refinement.

## Problem

Standard SfM systems like [[COLMAP]] and [[GLOMAP]] are designed for general monocular unstructured image collections and frequently fail or produce incomplete trajectories on autonomous driving data. They cannot leverage prior trajectory information (from SLAM or odometry), handle multi-camera vehicle rigs, or scale efficiently to the sequential structure of driving sequences. This makes them unsuitable for industrial applications requiring accurate offline pose refinement for downstream dense reconstruction.

## Method

CuSfM is built around a factor graph optimization framework with several key components:

1. **Non-Redundant Data Association**: Exploits coarse initial trajectory estimates (from visual SLAM like PyCuVSLAM or ORB-SLAM2) to construct a minimal yet comprehensive view graph, avoiding redundant correspondences. Two retrieval modes: radius-based search and view-graph based (using sequential adjacency and loop closures).

2. **GPU-Accelerated Feature Pipeline**: Uses TensorRT-accelerated ALIKED features for detection and description, providing enhanced robustness over SIFT-GPU at the cost of higher per-frame extraction time.

3. **Two-Stage Factor Graph Optimization**: First stage uses robust loss functions with reprojection errors, consecutive frame constraints, and absolute pose constraints. Second stage applies stringent outlier criteria without loss functions for high-precision results. Both stages use Ceres Solver with SPARSE_NORMAL_CHOLESKY.

4. **Extrinsic Refinement**: Treats multi-camera rigs as unified rigid bodies, decomposing each camera pose into vehicle pose and camera-to-vehicle transformation. Jointly optimizes vehicle poses and extrinsic calibration while preserving physical plausibility through prior constraints.

## Results

- **Runtime (KITTI)**: Total processing time 58.8s/100 frames (view-graph mode) vs. 346.2s for COLMAP and 102.4s for GLOMAP. Mapping phase alone is 20x faster than COLMAP.
- **SDG Dataset (NVIDIA simulation)**: ATE RMSE of 0.040m on Sequence 00 vs. COLMAP 0.406m (90.1% improvement) and GLOMAP 0.410m. COLMAP failed on 2 of 5 sequences.
- **KITTI**: Both COLMAP and GLOMAP fail to recover complete trajectories with consistent scale on monocular sequences. CuSfM successfully reconstructs all sequences using prior trajectory information.
- **Trajectory Refinement**: Consistently improves input PyCuVSLAM trajectories across all tested KITTI sequences.

## Why it matters

CuSfM addresses the practical gap between general-purpose SfM tools and industrial autonomous driving requirements. By leveraging prior trajectory estimates and multi-camera rig constraints, it provides a robust offline refinement system that bridges visual SLAM output and downstream dense reconstruction (3DGS, NeRF). The open-source PyCuSfM wrapper makes it accessible for research.

See [[gpu-native-sfm]] thread for the broader "Tier 1: accelerated classical SfM" story, and how CuSfM contrasts with [InstantSfM](zhong2026_instantsfm.md) on the generalization-specialization axis.

## Relation to prior work

- Competes directly with [[COLMAP]] and [[GLOMAP]] but tailored for sequential driving data with prior pose information.
- Uses PyCuVSLAM or ORB-SLAM2 for initial trajectory estimates, refining them through global optimization.
- Employs Ceres Solver for [[bundle-adjustment]], same backend as COLMAP/GLOMAP but with GPU-accelerated preprocessing.
- ALIKED features replace SIFT-GPU for more robust matching in driving scenarios.
- Downstream applications include [[3d-gaussian-splatting]] and [[NeRF]] dense reconstruction.

## Open questions / limitations

- Requires prior trajectory estimates; cannot operate from scratch on unordered image collections.
- COLMAP/GLOMAP comparison uses monocular input only, which disadvantages those systems (CuSfM leverages priors).
- Extrinsic refinement assumes rigid multi-camera rig; does not handle time-varying calibration.
- Limited evaluation on non-driving scenarios.

## References added to the wiki

- [[structure-from-motion]]
- [[COLMAP]]
- [[GLOMAP]]
- [[bundle-adjustment]]
- [[Ceres-Solver]]
- [[ORB-SLAM2]]
- [[3d-gaussian-splatting]]
- [[NeRF]]
- [[ALIKED]]
- [[factor-graph]]
