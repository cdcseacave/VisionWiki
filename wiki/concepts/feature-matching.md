---
title: Feature Matching
type: concept
tags: [correspondences, sfm, slam, keypoints, descriptors]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/edstedt2025_roma-v2.md
  - papers/murai2025_mast3r-slam.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/pataki2025_mp-sfm.md
  - papers/yu2025_cusfm.md
  - papers/zhong2026_instantsfm.md
status: stub
---

## What it is

Feature matching is the process of establishing pixel-level correspondences between two or more images of the same scene. It is the foundational input to Structure-from-Motion, SLAM, and many stereo methods. Classical approaches (SIFT, ORB) detect keypoints and match hand-crafted descriptors; modern learned methods (SuperGlue, LoFTR, RoMa) predict matches end-to-end, often densely.

## How it works

The classical pipeline involves: (1) keypoint detection, (2) descriptor extraction, (3) nearest-neighbor matching, and (4) geometric verification (RANSAC with epipolar geometry). Learned matchers replace some or all stages with neural networks. Dense matchers like RoMa v2 predict a full warp field between image pairs, bypassing keypoint detection entirely. The quality of feature matches directly determines the accuracy of downstream pose estimation and triangulation.

Since ~2023 the frontier has shifted to **frozen [[foundation-model|foundation-model]] features** — DINOv2/DINOv3 backbones drive RoMa v2 and the DUSt3R/MASt3R/VGGT matching heads. See [[foundation-features-for-geometry]] for the synthesis.

## Key references

- [Edstedt 2025](../papers/edstedt2025_roma-v2.md) · [pdf](../../papers/feature-matching/edstedt_2025_roma-v2.pdf)
- [Murai 2025](../papers/murai2025_mast3r-slam.md) · [pdf](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf)
- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
