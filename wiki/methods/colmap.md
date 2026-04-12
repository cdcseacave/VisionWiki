---
title: COLMAP
type: method
tags: [sfm, mvs, structure-from-motion, baseline]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/guo2025_ea-3dgs.md
  - papers/jang2025_pow3r.md
  - papers/jin2026_zipmap.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/park2023_camp.md
  - papers/pataki2025_mp-sfm.md
  - papers/tang2025_dronesplat.md
  - papers/yu2025_cusfm.md
  - papers/zhang2024_cameras-as-rays.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhao2025_diffusionsfm.md
  - papers/zhong2026_instantsfm.md
status: stub
---

## What it is

COLMAP is the de-facto standard open-source Structure-from-Motion (SfM) and Multi-View Stereo (MVS) pipeline, developed by Schoenberger & Frahm. It provides incremental and global SfM, dense stereo matching, and fusion. Nearly every 3D reconstruction paper benchmarks against COLMAP poses and/or point clouds.

## How it works

The SfM pipeline proceeds in stages: (1) feature extraction (SIFT), (2) exhaustive or vocabulary-tree feature matching, (3) geometric verification via RANSAC, (4) incremental reconstruction with repeated triangulation and bundle adjustment, and (5) optional dense MVS via patch-match stereo. COLMAP is known for robustness but can be slow on large-scale scenes due to its sequential, incremental design.

## Key references

- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf)
- [Yu 2025](../papers/yu2025_cusfm.md) · [pdf](../../papers/sfm-slam/yu_2025_cusfm.pdf)
- [Zhong 2026](../papers/zhong2026_instantsfm.md) · [pdf](../../papers/sfm-slam/zhong_2026_instantsfm.pdf)
- [Jin 2026](../papers/jin2026_zipmap.md) · [pdf](../../papers/sfm-slam/jin_2026_zipmap.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
