---
title: CLIP
type: method
tags: [vision-language, contrastive-learning, foundation-model]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/radford2021_clip.md]
status: draft
---

## What it is

CLIP (Radford et al. 2021, OpenAI) is a **dual-encoder vision-language model** trained by contrastive matching of 400M image-text pairs. It yields a shared image/text embedding space that supports zero-shot classification and open-vocabulary retrieval.

## How it works
- Image encoder (ViT or ResNet) + text encoder (Transformer).
- InfoNCE loss: pull matching image/text pairs together, push non-matches apart, within a batch.
- Inference: classify an image by comparing its embedding to text prompts ("a photo of a {class}").

## Why it matters in photogrammetry
- Enables **open-vocabulary 3D understanding**: distill CLIP features into radiance fields / 3DGS (LERF, OpenNeRF, 3DGS-language fields).
- Supplies semantic priors for segmentation-aware reconstruction and editing.
- Foundation backbone for many vision-language tasks downstream of 3D capture.

## Lineage
- CLIP (2021) → OpenCLIP (open replication) → EVA-CLIP, SigLIP (improved losses).

## Key references
- [Radford et al. 2021](../papers/radford2021_clip.md) · [pdf](../../papers/fundamentals/radford_2021_clip.pdf) — canonical CLIP paper, ICML 2021.
