---
title: DUSt3R
type: method
tags: [3d-reconstruction, feed-forward, two-view, transformer, pointmap]
created: 2026-04-12
updated: 2026-04-21
sources:
  - papers/edstedt2025_roma-v2.md
  - papers/elflein2026_vgg-t3.md
  - papers/jang2025_pow3r.md
  - papers/jin2026_zipmap.md
  - papers/leroy2024_mast3r.md
  - papers/li2025_megasam.md
  - papers/murai2025_mast3r-slam.md
  - papers/pataki2025_mp-sfm.md
  - papers/tang2025_dronesplat.md
  - papers/zhang2024_cameras-as-rays.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhang2025_loger.md
  - papers/zhao2025_diffusionsfm.md
  - papers/zhong2026_instantsfm.md
status: stub
---

## What it is

DUSt3R (Dense Unconstrained Stereo 3D Reconstruction), introduced by Wang et al. (CVPR 2024), is a feed-forward transformer that takes two uncalibrated images and directly predicts dense 3D point maps in a shared coordinate frame. It removes the need for camera intrinsics, explicit feature matching, or RANSAC, and serves as the foundation for a growing family of methods (MASt3R, PoW3R, ZipMap, InstantSfM, etc.).

## How it works

Two images are encoded by a shared ViT backbone, then cross-attended via a decoder that regresses per-pixel 3D coordinates and confidence. A global alignment step fuses pairwise predictions across multiple views via pointmap optimization. The model is trained on large-scale posed image datasets with a regression loss on 3D coordinates.

## Direct successor

- [[mast3r|MASt3R]] (Leroy et al. 2024) adds a dense-descriptor head + metric-scale loss variant on the same backbone, beating DUSt3R on every downstream benchmark (Map-free: +0.24 VCRE AUC; DTU zero-shot MVS: 4.6× Chamfer reduction). MASt3R initializes from the public DUSt3R checkpoint, so the two share weights at the encoder/decoder and diverge only at the heads. See the ingested paper page [Leroy et al. 2024](../papers/leroy2024_mast3r.md) and the four atomic ideas extracted there.

## Key references

- [Leroy 2024 (MASt3R)](../papers/leroy2024_mast3r.md) · [pdf](../../papers/feature-matching/leroy_2024_mast3r.pdf)
- [Jang 2025](../papers/jang2025_pow3r.md) · [pdf](../../papers/mvs-depth/jang_2025_pow3r.pdf)
- [Murai 2025](../papers/murai2025_mast3r-slam.md) · [pdf](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf)
- [Jin 2026](../papers/jin2026_zipmap.md) · [pdf](../../papers/sfm-slam/jin_2026_zipmap.pdf)
- [Zhong 2026](../papers/zhong2026_instantsfm.md) · [pdf](../../papers/sfm-slam/zhong_2026_instantsfm.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
