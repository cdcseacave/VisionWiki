---
title: Frozen DINOv3 + ViT-B multi-view transformer dense matcher (RoMa v2)
type: idea
source_paper: wiki/papers/edstedt2025_roma-v2.md
also_in: []

scope: stage-swap
stages: [feature-matching.task-head]
inputs: [image-pair, frozen-dinov3-features]
outputs: [dense-correspondence, per-pixel-2x2-precision-matrix]
assumptions: [frozen-backbone, image-pair, moderate-viewpoint-change]
requires_upstream_property: [dinov3-backbone-available]
requires_downstream_property: [consumer-accepts-dense-match-and-covariance]
learned_params: [matcher-transformer-weights, refiner-weights]
failure_modes: [textureless-sky-spurious-confidence, dinov3-16px-patch-limits-fine-scale]

requires: [dinov3-gram-anchoring_simeoni2025]
unlocks: []
co_requires: [roma-v2-predictive-covariance_edstedt2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [dense-matching, dinov3, transformer, cuda-kernel]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Alternating frame-wise / global attention in a ViT-B multi-view transformer on top of frozen DINOv3 features replaces RoMa's Gaussian-Process coarse matcher. Coarse matching runs at stride-4 (vs stride-14 in RoMa), requiring fewer refinement stages. A custom CUDA local-correlation kernel reduces refinement memory. Training: matcher 300k steps (batch 128), refiners 300k steps (batch 64) on wide+small-baseline dataset mix.

## Why it wins

SOTA pose-AUC on MegaDepth-1500 + ScanNet-1500 + WxBS + new SatAst benchmark. Stride-4 + custom CUDA → significantly faster than RoMa. Predictive covariance (paired idea) gives downstream geometry a principled per-match weight. Bundles with its own covariance head (`co_requires:`) — the matcher is useful alone, but the covariance output is what makes it load-bearing for downstream BA/SLAM.

## Preconditions & compatibility

Requires DINOv3 (`requires:` edge). DINOv3's 16-px patches limit finest-scale matching — a fixed constraint inherited from the backbone. Decoupled two-stage training prevents end-to-end gradient flow between matcher and refiners.

## Open questions

- Can TTT3R-style per-token learning rate (Bet #012) replace the two-stage decoupled training?
- Does stride-4 coarse matching generalize to larger backbones (ViT-L/g)?
