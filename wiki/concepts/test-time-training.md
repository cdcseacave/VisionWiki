---
title: Test-Time Training
type: concept
tags: [optimization, adaptation, self-supervised, emerging]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/elflein2026_vgg-t3.md
  - papers/jin2026_zipmap.md
  - papers/zhang2025_loger.md
status: stub
---

## What it is

Test-time training (TTT) is a paradigm where a pretrained model is further optimized on each test input (or batch of inputs) using a self-supervised objective, without access to ground-truth labels. In the context of 3D vision, TTT refines feed-forward predictions (e.g., from VGGT or DUSt3R) on the specific scene at hand, bridging the gap between fast feed-forward inference and per-scene optimization.

## How it works

After a feed-forward model produces an initial prediction, TTT defines a self-supervised loss on the test data itself -- commonly photometric consistency (rendering loss), geometric consistency (reprojection, depth), or cycle consistency. The model parameters (or a subset, such as adapter layers) are updated for a small number of gradient steps. This combines the generalization of large pretrained models with the precision of per-scene fitting. VGG-T3, ZipMap, and LoGeR all employ variants of this strategy.

## Key references

- [Elflein 2026](../papers/elflein2026_vgg-t3.md) · [pdf](../../papers/mesh-reconstruction/elflein_2026_vgg-t3.pdf)
- [Jin 2026](../papers/jin2026_zipmap.md) · [pdf](../../papers/sfm-slam/jin_2026_zipmap.pdf)
- [Zhang 2025](../papers/zhang2025_loger.md) · [pdf](../../papers/sfm-slam/zhang_2025_loger.pdf)
