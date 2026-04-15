---
title: "VPGS-SLAM: Voxel-based Progressive 3D Gaussian SLAM in Large-Scale Scenes"
type: paper
tags: [slam, 3d-gaussian-splatting, large-scale, loop-closure, voxel-representation, rgbd]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/sfm-slam/deng_2026_vpgs-slam.pdf
url: https://arxiv.org/abs/2505.18992
status: draft
---

📄 [Full paper](../../papers/sfm-slam/deng_2026_vpgs-slam.pdf) · [arXiv](https://arxiv.org/abs/2505.18992)

## TL;DR

VPGS-SLAM is a large-scale RGBD SLAM framework that uses a novel voxel-based progressive [[3d-gaussian-splatting]] representation with multiple submaps for compact scene representation. It introduces 2D-3D fusion camera tracking, Gaussian-based loop closure, and online distillation for submap fusion, enabling both indoor room-scale and outdoor city-scale SLAM with globally consistent maps.

## Problem

Existing 3DGS-based SLAM methods (SplaTAM, MonoGS, etc.) are constrained to small room-scale scenes due to three key challenges in large-scale environments: (a) redundant Gaussian ellipsoids causing memory explosion (>500MB for a single room), (b) accumulated pose drift from unreliable rendering-based tracking in challenging conditions (motion blur, exposure changes), and (c) lack of global consistency mechanisms for long sequences with revisits.

## Method

The system has three main modules:

1. **Voxel-based Progressive Scene Representation**: Multi-resolution voxel grids organize anchor points, each spawning neural Gaussians with learnable offsets. Attributes (opacity, color, rotation, scale) are decoded from anchor features and viewing direction via lightweight MLPs. Multiple submaps are dynamically allocated when the camera exceeds distance/rotation thresholds, keeping online memory bounded at O(N) per submap. Non-essential submaps are deactivated.

2. **2D-3D Fusion Camera Tracking**: Coarse-to-fine pose estimation combines 2D photometric loss (RGB + depth rendering) at coarse level with 3D voxel-based ICP at fine level. An adaptive strategy assesses rendering quality and falls back to pure 3D geometric tracking when 2D information is unreliable (outdoor scenes with illumination changes).

3. **2D-3D Gaussian Loop Closure**: Loop detection via NetVLAD descriptors, followed by pose graph optimization using both rendering loss and voxel ICP constraints. An online distillation method fuses overlapping submaps upon loop closure by aligning rendered RGB and depth images across submaps.

## Results

- **Replica**: ATE RMSE 0.21cm (best among all methods including NeRF-based); rendering PSNR 35.82, SSIM 0.984.
- **ScanNet**: ATE RMSE 7.6cm average across 8 sequences, outperforming SplaTAM (16.6cm), MonoGS (15.2cm), and Loop-Splat (8.4cm).
- **KITTI (LiDAR)**: ATE RMSE 1.40m average, the only 3DGS-based method that runs successfully on all 11 sequences. Outperforms NeRF-LOAM (5.88m) by ~75%.
- **KITTI/VKITTI2 (rendering)**: PSNR 21.37/25.45, SSIM 0.81/0.84, dramatically outperforming SplaTAM (14.68/18.29) and Loop-Splat (16.77/19.47).
- **Memory**: 2-3x lower GPU usage than SplaTAM; 6x lower map memory through voxelized representation.

## Why it matters

VPGS-SLAM is one of the first 3DGS-based SLAM systems to scale from room-level to city-level scenes, addressing a critical gap for autonomous driving and large-scale robotics. The progressive submap strategy with online distillation provides a practical framework for managing the memory-accuracy tradeoff inherent in explicit Gaussian representations.

## Pipeline contribution

- **Multi-resolution voxel-grid anchors + learned per-anchor neural Gaussians (N1)** — attributes decoded from anchor features via MLPs; dynamic submap allocation keeps online memory O(N) per submap. candidate thread: [[radiance-field-evolution]] · stage: *large-scale 3DGS representation* · replaces/augments: *global 3DGS with unbounded memory* · expected gain: 6× lower map memory, 2–3× lower GPU usage than SplaTAM.
- **2D-3D fusion camera tracking (N2)** — coarse 2D photometric + fine 3D voxel ICP with adaptive fallback. candidate thread: *3DGS SLAM* · stage: *per-frame tracking* · replaces/augments: *rendering-only tracking* · expected gain: robust in challenging conditions (motion blur, exposure) where rendering-only tracking drifts.
- **2D-3D Gaussian loop closure + online-distillation submap fusion (N3)** — NetVLAD loop detection; pose graph with rendering+ICP constraints; submap fusion aligns rendered RGB/depth. candidate thread: *3DGS SLAM* · stage: *global consistency* · replaces/augments: *no loop closure in SplaTAM/MonoGS* · expected gain: globally-consistent city-scale 3DGS SLAM.
- **Role**: with [[lin2024_vastgaussian]] (offline city-scale) and [[murai2025_mast3r-slam]] (calibration-free monocular), VPGS-SLAM completes the "3DGS can go large-scale" story in [[radiance-field-evolution]]'s scaling subtree.

## Relation to prior work

- Extends [[3d-gaussian-splatting]] SLAM methods (SplaTAM, MonoGS, [[Gaussian-SLAM]], Loop-Splat) to large-scale scenes.
- Progressive submap approach inspired by [[PLGSLAM]] and similar neural implicit submap methods.
- Voxel-based anchor representation draws from Scaffold-GS and [[octree-gs]].
- Loop closure improves on Loop-Splat by combining 2D rendering loss with 3D voxel ICP.
- Competes with NeRF-based SLAM (NICE-SLAM, Go-SLAM, [[NeRF-LOAM]]) and LiDAR methods (KISS-ICP, PIN-SLAM).

## Open questions / limitations

- Requires RGBD input (depth sensor or LiDAR); not applicable to monocular-only settings.
- Uses DepthLab for dense depth estimation in outdoor scenes, adding a dependency on external depth completion.
- Real-time performance not explicitly benchmarked for outdoor sequences.
- Global optimization limited to loop closure; no full global [[bundle-adjustment]].

## References added to the wiki

- [[3d-gaussian-splatting]]
- [[SLAM]]
- [[loop-closure]]
- [[bundle-adjustment]]
- [[ICP]]
- [[NeRF-LOAM]]
- [[SplaTAM]]
- [[KISS-ICP]]
- [[pose-graph-optimization]]
- [[NetVLAD]]
