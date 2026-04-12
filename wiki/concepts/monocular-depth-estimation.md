---
title: Monocular Depth Estimation
type: concept
tags: [depth, single-view, prior, foundation-model]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/deng2026_vpgs-slam.md
  - papers/jang2025_pow3r.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/li2025_geosvr.md
  - papers/li2025_megasam.md
  - papers/murai2025_mast3r-slam.md
  - papers/pataki2025_mp-sfm.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhong2026_instantsfm.md
status: stub
---

## What it is

Monocular depth estimation predicts a dense depth map from a single RGB image. Modern methods (Depth Anything, Marigold, Metric3D) leverage large-scale pretraining to produce robust relative or metric depth priors. These priors are increasingly used as initialization or regularization in SfM, SLAM, and novel-view synthesis pipelines to resolve scale ambiguity and improve geometry in textureless regions.

## How it works

A deep network (typically a vision transformer encoder with a convolutional decoder) is trained on diverse image-depth pairs to regress per-pixel depth. Training data comes from sensor depth (LiDAR, structured light), synthetic rendering, and pseudo-labels from stereo/SfM. The output is often affine-invariant (relative depth up to scale and shift). Downstream systems align the monocular prior to multi-view constraints via scale-shift fitting or joint optimization.

## Key references

- [Li 2025 (MegaSaM)](../papers/li2025_megasam.md) · [pdf](../../papers/sfm-slam/li_2025_megasam.pdf)
- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf)
- [Zhong 2026](../papers/zhong2026_instantsfm.md) · [pdf](../../papers/sfm-slam/zhong_2026_instantsfm.pdf)
- [Jang 2025](../papers/jang2025_pow3r.md) · [pdf](../../papers/mvs-depth/jang_2025_pow3r.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
