---
title: CLIP distillation into per-Gaussian latents via scene-specific autoencoder
type: idea
source_paper: wiki/papers/qin2024_langsplat.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.per-primitive-feature-storage]
collapses: []
splits_into: []
rewrites: {}

inputs: [per-view-clip-embeddings-at-sam-mask-scales, 3d-gaussians]
outputs: [per-gaussian-compressed-latent, query-time-decoder]
assumptions: [static-scene, posed-input, clip-features-meaningful-per-scene, commercial-license-blocked-by-3dgs-base]
requires_upstream_property: [clip-features-per-view, sam-mask-hierarchy-available]
requires_downstream_property: [query-time-can-decode-per-gaussian-latent]
learned_params: [autoencoder-encoder-weights, autoencoder-decoder-weights, gaussian.language-latent]
failure_modes: [no-cross-scene-transfer, clip-semantic-ceiling-on-fine-grained-classes]

requires: []
unlocks: []
co_requires: [sam-mask-hierarchy-supervision_qin2024]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, clip, language-field, per-scene-autoencoder]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Raw CLIP embeddings (512-D) are too memory-heavy to store per Gaussian at 1M+ primitives. A scene-specific autoencoder is trained on the scene's CLIP feature distribution to compress 512-D → low-D (typically 3–16 dim). Each Gaussian stores the low-D latent; at query time the decoder expands the latent back to the CLIP space for cosine similarity with the query embedding. Per-scene fitting means the latent space is optimized to the distribution this scene's objects inhabit — unused CLIP subspaces are thrown away.

## Why it wins

Memory: per-Gaussian latent is ~30× smaller than raw CLIP. Speed: 199× faster than LERF at 1440×1080 — the autoencoder compression is load-bearing for this, combined with 3DGS-vs-NeRF rasterization. Sharper boundaries come not from the autoencoder but from the bundled SAM hierarchy supervision (hence `co_requires:`).

## Preconditions & compatibility

The autoencoder is *per-scene*; no cross-scene transfer. If cross-scene zero-shot matters, this idea is the wrong choice — op:zero-shot-3dgs (CLIP-GS's scene-level contrastive alignment) is the alternative. Compatible with any 2D feature source — Bet #014 explores RADIOv2.5 as a drop-in for CLIP.

## Open questions

- Does re-adding DINOv3 (LangSplat explicitly dropped DINOv2) restore useful spatial coherence that the SAM hierarchy doesn't supply?
- Optimal latent dim is a free hyperparameter.
