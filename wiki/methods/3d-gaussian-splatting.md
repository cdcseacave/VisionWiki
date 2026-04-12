---
title: 3D Gaussian Splatting
type: method
tags: [radiance-fields, real-time-rendering, point-based, differentiable]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/deng2026_vpgs-slam.md
  - papers/guo2025_ea-3dgs.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/li2025_geosvr.md
  - papers/li2025_va-gs.md
  - papers/radl2025_sof.md
  - papers/radl2026_confidence-mesh-3dgs.md
  - papers/sun2025_sparse-voxels-rasterization.md
  - papers/tang2025_dronesplat.md
  - papers/xie2025_gauss-mi.md
  - papers/yu2025_cusfm.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhu2025_gs-discretized-sdf.md
status: stub
---

## What it is

3D Gaussian Splatting (3DGS) is a point-based scene representation that models a scene as a collection of anisotropic 3D Gaussians, each parameterized by position, covariance, opacity, and view-dependent color (typically via spherical harmonics). Introduced by Kerbl et al. (SIGGRAPH 2023), it achieves real-time novel-view synthesis at quality competitive with NeRF while being orders of magnitude faster to render.

## How it works

Each Gaussian is projected ("splatted") onto the image plane via a differentiable tile-based rasterizer. Alpha-compositing of the sorted splats produces the final image. Training optimizes Gaussian parameters via photometric loss, with adaptive densification and pruning to control the point count. The explicit nature of the representation enables direct editing, fast rendering, and straightforward integration with downstream tasks like SLAM and mesh extraction.

## Key references

- [Guo 2025](../papers/guo2025_ea-3dgs.md) · [pdf](../../papers/radiance-fields/guo_2025_ea-3dgs.pdf)
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md) · [pdf](../../papers/radiance-fields/kim_2025_multiview-geometric-gs.pdf)
- [Radl 2026](../papers/radl2026_confidence-mesh-3dgs.md) · [pdf](../../papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf)
- [Sun 2025](../papers/sun2025_sparse-voxels-rasterization.md) · [pdf](../../papers/radiance-fields/sun_2025_sparse-voxels-rasterization.pdf)
- [Tang 2025](../papers/tang2025_dronesplat.md) · [pdf](../../papers/radiance-fields/tang_2025_dronesplat.pdf)
- [Deng 2026](../papers/deng2026_vpgs-slam.md) · [pdf](../../papers/sfm-slam/deng_2026_vpgs-slam.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
