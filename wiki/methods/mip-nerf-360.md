---
title: Mip-NeRF 360
type: method
tags: [radiance-fields, nerf, unbounded-scenes, anti-aliasing, proposal-network, distortion-regularizer]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/barron2022_mip-nerf-360.md, papers/barron2023_zip-nerf.md, papers/kim2025_multiview-geometric-gs.md, papers/sun2025_sparse-voxels-rasterization.md]
status: stable
---

## What it is

Unbounded-scene radiance-field method (Barron et al., CVPR 2022) that extends Mip-NeRF with three coordinated ideas: a non-linear scene contraction, a proposal network trained via online distillation, and a distortion-based regularizer that suppresses floaters and background collapse. See [Barron et al. 2022](../papers/barron2022_mip-nerf-360.md) for full derivation.

## How it works

1. **Scene contraction**: maps all of $\mathbb{R}^3$ into a ball of radius 2, with distant points parameterized proportionally to disparity (NDC generalized to 360° captures). Mip-NeRF's Gaussian cone-casting is lifted through the contraction via a linearized EKF-style pushforward.
2. **Proposal MLP + online distillation**: a small, cheap proposal MLP is trained to output weight histograms that bound a larger NeRF MLP's weights (not to render images directly). Many proposal evaluations + resamples, one NeRF-MLP evaluation per ray — effectively a much higher-capacity NeRF at moderate extra cost.
3. **Distortion loss**: a per-ray regularizer that concentrates density on surfaces rather than spreading it across the ray — directly suppresses floaters and background collapse that plague NeRF on unbounded scenes.

## Benchmark legacy

The paper introduced the **Mip-NeRF 360 dataset** — 9 unbounded scenes — which has become the canonical novel-view-synthesis benchmark for outdoor/unbounded captures. Nearly every subsequent 3DGS, sparse-voxel, and NeRF paper in [[radiance-field-evolution]] reports MipNeRF360-dataset PSNR/SSIM/LPIPS.

## Strengths

- First NeRF variant that worked cleanly on unbounded 360° scenes.
- Distortion regularizer is a principled rather than empirical fix — influences later surface-focused methods.
- 57% MSE reduction vs. Mip-NeRF; still the implicit-NeRF quality reference before [[zip-nerf|Zip-NeRF]] extended it.

## Limitations

- Slow: ~1 day on 32 TPU-v2 per scene.
- Requires known poses/intrinsics (no joint camera optimization — see [[park2023_camp|CamP]] for that).
- Contraction assumes 360° *rotation* around a point; translation-heavy city-scale captures need partitioning instead (see VastGaussian in [[radiance-field-evolution]]).

## Variants and lineage

- Predecessor: Mip-NeRF (Barron 2021) — anti-aliased cone-casting for bounded scenes.
- Direct successor: [[zip-nerf|Zip-NeRF]] (Barron 2023) — adds hash-grid speed on top of the contraction + proposal network.
- Stacks with [[park2023_camp|CamP]] (Park 2023) — adds joint camera preconditioning.
- Displaced on outdoor scenes by explicit [[3d-gaussian-splatting|3DGS]] variants (faster at comparable quality).

## Key references

- [Barron et al. 2022](../papers/barron2022_mip-nerf-360.md) — the method paper.
- [Zip-NeRF (Barron et al. 2023)](../papers/barron2023_zip-nerf.md) — hash-grid successor.
