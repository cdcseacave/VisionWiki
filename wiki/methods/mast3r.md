---
title: MASt3R
type: method
tags: [3d-reconstruction, feature-matching, feed-forward, transformer]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/edstedt2025_roma-v2.md
  - papers/elflein2026_vgg-t3.md
  - papers/jang2025_pow3r.md
  - papers/mao2025_spatiallm.md
  - papers/murai2025_mast3r-slam.md
  - papers/pataki2025_mp-sfm.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhao2025_diffusionsfm.md
status: stub
---

## What it is

MASt3R (Matching And Stereo 3D Reconstruction) extends DUSt3R by jointly predicting dense 3D point maps and local feature descriptors suitable for matching. Introduced by Leroy et al. (2024), it unifies stereo reconstruction with feature matching in a single feed-forward pass, enabling robust two-view pose estimation without a separate matcher.

## How it works

MASt3R augments the DUSt3R architecture with an additional head that outputs dense local descriptors alongside the 3D point maps. These descriptors can be matched across views using nearest-neighbor search, providing correspondences that are geometrically grounded. The combined point-map and matching output enables fast, robust pose estimation and serves as a backbone for downstream SLAM systems (MASt3R-SLAM) and SfM pipelines.

## Key references

- [Murai 2025](../papers/murai2025_mast3r-slam.md) · [pdf](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf)
- [Edstedt 2025](../papers/edstedt2025_roma-v2.md) · [pdf](../../papers/feature-matching/edstedt_2025_roma-v2.pdf)
- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
