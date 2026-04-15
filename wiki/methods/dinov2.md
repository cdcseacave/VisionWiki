---
title: DINOv2
type: method
tags: [self-supervised-learning, vision-transformer, foundation-model, features]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/oquab2023_dinov2.md]
status: draft
---

## What it is

DINOv2 is a **self-supervised vision foundation model** (Meta, 2023) producing dense image features that transfer strongly to downstream tasks without fine-tuning, including depth estimation, segmentation, and correspondence.

## How it works

- Self-distillation with no labels on a curated 142M-image dataset.
- Student/teacher [[vision-transformer|ViT]] trained with DINO loss + iBOT patch-level loss.
- Produces patch tokens that are semantically discriminative and geometrically consistent.

## Why it matters in photogrammetry

- Feature backbone for feed-forward SfM / MVS methods ([[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]], CUT3R).
- Supplies dense descriptors for learned matching and monocular depth ([[monocular-depth-estimation]]).
- Establishes the "frozen foundation backbone + task head" pattern now standard in 2025–2026 3D vision. See [[foundation-features-for-geometry]] thread.
- Part of the [[foundation-model]] family; produced by [[self-supervised-learning|vision-only SSL]].

## Variants & lineage
- DINO (2021) → DINOv2 (2023) → [DINOv3](../papers/simeoni2025_dinov3.md) (2025) — Gram anchoring + post-hoc resolution/text alignment.

## Key references
- [Oquab et al. 2023](../papers/oquab2023_dinov2.md) · [pdf](../../papers/fundamentals/oquab_2023_dinov2.pdf) — canonical DINOv2 paper.
