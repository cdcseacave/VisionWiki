---
title: Contrastive dual-encoder for image-text alignment (CLIP)
type: idea
source_paper: wiki/papers/radford2021_clip.md
also_in: []

scope: new-paradigm
stages: [open-vocab-2d.semantic-alignment]
inputs: [paired-image-text-corpus-400M]
outputs: [shared-image-text-embedding-space, zero-shot-prompt-classifier]
assumptions: [web-scale-contrastive-data-available, image-level-training-OK]
requires_upstream_property: [vit-or-cnn-image-encoder, text-encoder]
requires_downstream_property: [consumer-does-cosine-similarity-in-shared-space]
learned_params: [image-encoder-weights, text-encoder-weights, learned-temperature]
failure_modes: [spatially-weak-image-level-only, compositional-prompts-fail]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [clip, contrastive, dual-encoder, semantic-alignment, foundational]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Symmetric InfoNCE over the N×N image-text similarity matrix with learned temperature. Produces a shared embedding space in which cosine similarity means "this image matches this caption." Zero-shot classification at inference: embed all candidate class names as text, embed the image, return the argmax.

## Why it wins

Zero-shot mIoU transfer across 30+ datasets. The semantic-alignment leg of Trident (+4.2% over prior SOTA in Shi 2024). Foundational for every CLIP-lifted 3D method (LangSplat distills CLIP per-Gaussian).

## Preconditions & compatibility

**Excluded from** [[foundation-features-for-geometry]] — CLIP is image-level and text-aligned but spatially weak. SigLIP (pairwise sigmoid loss) is the natural drop-in successor; EVA-CLIP adds MIM pretraining. Compositional prompts ("red chair without armrests") remain weak.

## Open questions

- Image-level training limits spatial precision — is there a spatial-aware contrastive variant that keeps zero-shot transfer?
