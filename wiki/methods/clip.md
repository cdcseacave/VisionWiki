---
title: CLIP
type: method
tags: [vision-language, contrastive-learning, foundation-model]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/radford2021_clip.md, wiki/papers/shi2024_open-vocab-segmentation.md, wiki/papers/qin2024_langsplat.md, wiki/papers/jiao2025_clip-gs.md]
status: draft
---

## What it is

CLIP (Radford et al. 2021, OpenAI) is a **dual-encoder vision-language model** trained by contrastive matching of 400M image-text pairs. It yields a shared image/text embedding space that supports zero-shot classification and open-vocabulary retrieval.

Part of the [[foundation-model]] family; pairs with [[dinov2|DINOv2]] and [[sam|SAM]] in the [[open-vocabulary-segmentation]] [[open-vocab-2d-composition|composition]] pattern.

## How it works
- Image encoder (ViT or ResNet) + text encoder (Transformer).
- InfoNCE loss: pull matching image/text pairs together, push non-matches apart, within a batch.
- Inference: classify an image by comparing its embedding to text prompts ("a photo of a {class}").

## Why it matters in photogrammetry
- Enables **open-vocabulary 3D understanding**. See [[lifting-foundation-models-to-3d]]:
  - [[qin2024_langsplat|LangSplat]] — distills CLIP into per-Gaussian latents, replacing NeRF-based LERF.
  - [[jiao2025_clip-gs|CLIP-GS]] — aligns whole-scene 3DGS embeddings with CLIP image/text for zero-shot 3D retrieval.
- Supplies semantic priors for segmentation-aware reconstruction and editing.
- Composes with DINO + SAM in open-vocabulary-segmentation pipelines ([[shi2024_open-vocab-segmentation|Trident]]).
- Foundation backbone for many vision-language tasks downstream of 3D capture.

## Lineage
- CLIP (2021) → OpenCLIP (open replication) → EVA-CLIP, SigLIP (improved losses).

## Key references
- [Radford et al. 2021](../papers/radford2021_clip.md) · [pdf](../../papers/fundamentals/radford_2021_clip.pdf) — canonical CLIP paper, ICML 2021.
