---
title: Bundle Adjustment
type: concept
tags: [optimization, sfm, slam, nonlinear-least-squares]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/deng2026_vpgs-slam.md
  - papers/jin2026_zipmap.md
  - papers/li2025_megasam.md
  - papers/murai2025_mast3r-slam.md
  - papers/park2023_camp.md
  - papers/pataki2025_mp-sfm.md
  - papers/yu2025_cusfm.md
  - papers/zhang2024_cameras-as-rays.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhang2025_loger.md
  - papers/zhao2025_diffusionsfm.md
  - papers/zhong2026_instantsfm.md
status: stub
---

## What it is

Bundle adjustment (BA) is the gold-standard nonlinear optimization for jointly refining 3D structure and camera parameters by minimizing reprojection error. The term "bundle" refers to the bundle of rays connecting each 3D point to the cameras that observe it. BA is the final (and often most important) stage of any SfM or SLAM pipeline.

## How it works

BA formulates a nonlinear least-squares problem: minimize the sum of squared reprojection errors across all observations. The Jacobian has a characteristic sparse block structure (points vs. cameras) that is exploited by the Schur complement trick, reducing the system to a much smaller camera-only problem. Solvers like Ceres or g2o implement Levenberg-Marquardt with this sparse structure. Modern variants include global BA (all frames at once), local BA (sliding window), and differentiable BA layers used inside learned systems.

## Key references

- [Yu 2025](../papers/yu2025_cusfm.md) · [pdf](../../papers/sfm-slam/yu_2025_cusfm.pdf)
- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf)
- [Zhong 2026](../papers/zhong2026_instantsfm.md) · [pdf](../../papers/sfm-slam/zhong_2026_instantsfm.pdf)
- [Deng 2026](../papers/deng2026_vpgs-slam.md) · [pdf](../../papers/sfm-slam/deng_2026_vpgs-slam.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
