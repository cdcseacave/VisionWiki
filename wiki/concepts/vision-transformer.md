---
title: Vision Transformer (ViT)
type: concept
tags: [architecture, backbone, attention, transformer]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/chebbi2025_multiview-dense-matching.md
  - papers/edstedt2025_roma-v2.md
  - papers/guedon2025_milo.md
  - papers/heinrich2025_radiov25.md
  - papers/jang2025_pow3r.md
  - papers/radl2026_confidence-mesh-3dgs.md
  - papers/shi2024_open-vocab-segmentation.md
  - papers/zhang2024_cameras-as-rays.md
status: stub
---

## What it is

The Vision Transformer (ViT), introduced by Dosovitskiy et al. (ICLR 2021), applies the transformer architecture to image recognition by splitting an image into fixed-size patches, linearly embedding them, and processing the sequence with standard transformer encoder layers. ViT and its variants (DINOv2, DPT, CroCo) serve as the backbone encoder for most modern feed-forward 3D methods including DUSt3R, MASt3R, VGGT, and many dense matchers.

## How it works

An image is divided into non-overlapping patches (typically 16x16 or 14x14 pixels). Each patch is flattened and linearly projected to a token embedding. Positional embeddings are added, and the sequence is processed through L layers of multi-head self-attention and feed-forward blocks. The output tokens carry rich, spatially-aware features. For dense prediction tasks, decoder heads or feature pyramid networks process these tokens into per-pixel outputs. Pretrained ViT backbones (especially DINOv2) provide strong geometric priors that transfer well to 3D vision tasks.

## Key references

- [Jang 2025](../papers/jang2025_pow3r.md) · [pdf](../../papers/mvs-depth/jang_2025_pow3r.pdf)
- [Heinrich 2025](../papers/heinrich2025_radiov25.md) · [pdf](../../papers/fundamentals/heinrich_2025_radiov25.pdf)
- [Edstedt 2025](../papers/edstedt2025_roma-v2.md) · [pdf](../../papers/feature-matching/edstedt_2025_roma-v2.pdf)
- [Zhang 2024](../papers/zhang2024_cameras-as-rays.md) · [pdf](../../papers/pose-estimation/zhang_2024_cameras-as-rays.pdf)
