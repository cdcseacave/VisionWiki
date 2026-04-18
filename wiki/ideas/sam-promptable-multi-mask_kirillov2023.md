---
title: Promptable segmentation with ambiguity-aware multi-mask head (SAM)
type: idea
source_paper: wiki/papers/kirillov2023_sam.md
also_in: []

scope: new-paradigm
stages: [open-vocab-2d.mask-boundaries]
inputs: [image, point-or-box-or-text-prompt]
outputs: [three-candidate-masks-with-confidences]
assumptions: [web-scale-masks-available-sa-1b]
requires_upstream_property: [vit-h-image-encoder]
requires_downstream_property: [consumer-picks-best-mask-or-uses-all-three]
learned_params: [image-encoder-weights, prompt-decoder-weights]
failure_modes: [low-level-similarity-over-semantic-similarity-oversegments-subparts]

requires: []
unlocks: [langsplat-per-scene-autoencoder_qin2024, gaussian-grouping-identity-encoding_ye2024]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sam, promptable-segmentation, foundational]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Heavyweight ViT-H image encoder runs **once per image**; lightweight decoder runs per prompt (point / box / text). The decoder emits up to 3 valid masks + confidence scores — the ambiguity-aware head handles the "is this click on the person or the shirt?" case by returning both interpretations with confidences.

## Why it wins

Clean boundaries snapped to any spatially-coherent region score — the mask-quality leg of Trident. Encode-once / decode-per-prompt amortizes cost across many queries, making per-Gaussian identity distillation tractable (Gaussian Grouping, LangSplat).

## Preconditions & compatibility

SA-1B data engine (three-stage assisted → semi → fully automatic) is the underlying data recipe that SAM 3's SA-Co extends. **Superseded-by** [[sam3-native-video-ids_carion2026]] in the unified-promptable-concept role; SAM still dominant for pure segmentation without concept labels.

## Open questions

- SAM's features capture low-level visual similarity, not semantic — over-segments objects with distinct sub-parts.
