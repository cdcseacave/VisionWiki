---
title: VGGT (Visual Geometry Grounded Transformer)
type: method
tags: [3d-reconstruction, feed-forward, transformer, multi-task, geometry]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/edstedt2025_roma-v2.md
  - papers/elflein2026_vgg-t3.md
  - papers/jin2026_zipmap.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhang2025_loger.md
status: stub
---

## What it is

VGGT (Wang et al., 2025) is a feed-forward transformer that takes an arbitrary number of unposed images and jointly predicts camera parameters, point maps, depth, and 3D point tracks in a single forward pass. It generalizes the two-view DUSt3R paradigm to multi-view, producing geometry-grounded outputs without iterative optimization.

## How it works

Images are encoded with a ViT backbone and processed through alternating self-attention (within each view) and cross-attention (across views) layers. Multiple prediction heads decode camera extrinsics/intrinsics, per-pixel depth, dense point maps, and point tracks. The model is trained end-to-end on large-scale datasets with multi-task losses. Its single-pass design makes it attractive as a fast initializer for downstream refinement (e.g., test-time training in LoGeR and ZipMap).

## Key references

- [Elflein 2026](../papers/elflein2026_vgg-t3.md) · [pdf](../../papers/mesh-reconstruction/elflein_2026_vgg-t3.pdf)
- [Jin 2026](../papers/jin2026_zipmap.md) · [pdf](../../papers/sfm-slam/jin_2026_zipmap.pdf)
- [Zhang 2025 (LoGeR)](../papers/zhang2025_loger.md) · [pdf](../../papers/sfm-slam/zhang_2025_loger.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
