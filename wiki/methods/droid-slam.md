---
title: DROID-SLAM
type: method
tags: [slam, dense, visual-odometry, deep-learning]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/li2025_megasam.md
  - papers/murai2025_mast3r-slam.md
  - papers/zhang2025_loger.md
status: stub
---

## What it is

DROID-SLAM (Teed & Deng, NeurIPS 2021) is a deep visual SLAM system that performs dense bundle adjustment over optical-flow-based correspondences. It operates on monocular or stereo video and produces accurate camera trajectories and dense depth maps. It is a common baseline for learned SLAM methods.

## How it works

DROID-SLAM maintains a set of keyframes with associated depth and pose estimates. A recurrent network (based on RAFT) iteratively updates optical flow between keyframe pairs, and a differentiable Dense Bundle Adjustment (DBA) layer jointly optimizes poses and depth maps to be consistent with the predicted flow. The system selects keyframes adaptively and runs the update operator in a loop, refining estimates over time.

## Key references

- [Murai 2025](../papers/murai2025_mast3r-slam.md) · [pdf](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf)
- [Li 2025](../papers/li2025_megasam.md) · [pdf](../../papers/sfm-slam/li_2025_megasam.pdf)
- [Zhang 2025](../papers/zhang2025_loger.md) · [pdf](../../papers/sfm-slam/zhang_2025_loger.pdf)
