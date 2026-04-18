---
title: DINO+iBOT patch-level self-supervised pretraining (DINOv2)
type: idea
source_paper: wiki/papers/oquab2023_dinov2.md
also_in: []

scope: new-paradigm
stages: [foundation-features.pretraining]
inputs: [curated-142m-image-dataset]
outputs: [frozen-vit-backbone-with-dense-patch-features]
assumptions: [ssl-pretraining-budget, distillation-infrastructure]
requires_upstream_property: []
requires_downstream_property: [consumer-uses-frozen-patch-features]
learned_params: [vit-backbone-weights]
failure_modes: [dense-features-silent-collapse-at-long-schedules-on-vit-g]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [dinov2, self-supervised, vit, foundational]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Combined DINO image-level + iBOT patch-level self-supervised losses on a curated 142M image dataset. KoLeo regularizer makes frozen features linearly separable; distillation recipe produces smaller students from ViT-g/14 without quality loss. No text supervision — trades zero-shot classification for strong dense features.

## Why it wins

First self-supervised ViT that beats task-trained descriptors on multiple downstream geometry heads without fine-tuning. Generalizes to textureless / illumination-varying scenes where SIFT collapses. The frozen-backbone recipe — "ViT pretrain once, train small heads for each task" — becomes the default pattern for 2024+ geometry (RoMa v2, DUSt3R, MASt3R, VGGT, Metric3Dv2).

## Preconditions & compatibility

**Superseded by** [[dinov3-gram-anchoring_simeoni2025]] for new pipelines (which fixes the ViT-g long-schedule collapse), but DINOv2 remains widely deployed. Still the teacher in RADIOv2.5's multi-teacher distillation.

## Open questions

- Why does ViT-g collapse on long schedules? DINOv3's Gram anchoring fixes the symptom, not the cause.
