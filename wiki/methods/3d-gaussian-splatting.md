---
title: 3D Gaussian Splatting
type: method
tags: [radiance-fields, real-time-rendering, point-based, differentiable]
created: 2026-04-12
updated: 2026-04-15
sources:
  - papers/ye2024_gaussian-grouping.md
  - papers/qin2024_langsplat.md
  - papers/chen2024_pgsr.md
  - papers/jiao2025_clip-gs.md
  - papers/bao2025_seg-wild.md
  - papers/kim2026_gauss-explorer.md
  - papers/deng2026_vpgs-slam.md
  - papers/guo2025_ea-3dgs.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/li2025_geosvr.md
  - papers/li2025_va-gs.md
  - papers/lin2024_vastgaussian.md
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

## Scaling to large scenes

Vanilla 3DGS breaks on city/aerial-scale captures: a single GPU OOMs with the required tens of millions of Gaussians, and even when memory fits, densification starvation produces blurry results with floaters caused by cross-image appearance drift. Several complementary directions:

- **Spatial partitioning** — [VastGaussian (Lin 2024)](../papers/lin2024_vastgaussian.md) divides the scene into cells on the ground plane, trains each cell in parallel with an *airspace-aware* visibility criterion for camera assignment, then merges. Pairs with a decoupled per-image appearance module that transforms training targets (not the representation), so Gaussians absorb only view-consistent structure and the module is discarded at inference.
- **Compression** — [EA-3DGS (Guo 2025)](../papers/guo2025_ea-3dgs.md) uses tetrahedral mesh init + codebook quantization for 5.2× compression on outdoor scenes.
- **Incremental SLAM** — [VPGS-SLAM (Deng 2026)](../papers/deng2026_vpgs-slam.md) voxel-based progressive training with loop closure, avoiding a priori scene partitioning.

These are orthogonal and combinable.

## Foundation-model lifting onto 3DGS

A parallel 2024–2026 direction augments each Gaussian with features distilled or supervised from 2D foundation models, enabling concept-level querying, segmentation, and editing:

- **SAM-lifted identities** — [Gaussian Grouping (Ye 2024)](../papers/ye2024_gaussian-grouping.md) adds a per-Gaussian Identity Encoding supervised by SAM masks; enables object removal/inpainting/recomposition.
- **SAM-lifted, in-the-wild** — [Seg-Wild (Bao 2025)](../papers/bao2025_seg-wild.md) extends identity embeddings + adds the Spiky Gaussian Cutter for internet photo collections with transient occluders.
- **CLIP-lifted language fields** — [LangSplat (Qin 2024)](../papers/qin2024_langsplat.md) distills CLIP into per-Gaussian latents with SAM-hierarchy supervision; 199× faster than LERF at 1440×1080.
- **CLIP-aligned 3DGS encoder** — [CLIP-GS (Jiao 2025)](../papers/jiao2025_clip-gs.md) tokenizes 3DGS and contrastively aligns it with CLIP at the scene level for zero-shot 3D retrieval/classification.
- **VLM reasoning on 3DGS** — [GaussExplorer (Kim 2026)](../papers/kim2026_gauss-explorer.md) treats 3DGS as a queryable spatial database for compositional embodied reasoning.

See the [[lifting-foundation-models-to-3d]] thread for the synthesis.

## Planar/surface-oriented 3DGS

- [PGSR (Chen 2024)](../papers/chen2024_pgsr.md) — flatten Gaussians into planes + unbiased depth rendering + multi-view geometric regularization; SOTA 3DGS mesh quality without external priors. See [[gaussian-to-mesh-pipelines]].

## Key references

- [Guo 2025](../papers/guo2025_ea-3dgs.md) · [pdf](../../papers/radiance-fields/guo_2025_ea-3dgs.pdf)
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md) · [pdf](../../papers/radiance-fields/kim_2025_multiview-geometric-gs.pdf)
- [Lin 2024 (VastGaussian)](../papers/lin2024_vastgaussian.md) · [pdf](../../papers/radiance-fields/lin_2024_vastgaussian.pdf)
- [Radl 2026](../papers/radl2026_confidence-mesh-3dgs.md) · [pdf](../../papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf)
- [Sun 2025](../papers/sun2025_sparse-voxels-rasterization.md) · [pdf](../../papers/radiance-fields/sun_2025_sparse-voxels-rasterization.pdf)
- [Tang 2025](../papers/tang2025_dronesplat.md) · [pdf](../../papers/radiance-fields/tang_2025_dronesplat.pdf)
- [Deng 2026](../papers/deng2026_vpgs-slam.md) · [pdf](../../papers/sfm-slam/deng_2026_vpgs-slam.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
