---
title: Large-chunk TTT fast-weight MLP replacing global attention (ZipMap)
type: idea
source_paper: wiki/papers/jin2026_zipmap.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.attention-mechanism]
inputs: [multi-frame-tokens]
outputs: [O(N)-attention-output]
assumptions: [vggt-style-block-architecture-available, muon-optimizer-available]
requires_upstream_property: [dinov2-or-croco-tokenizer]
requires_downstream_property: [prediction-heads-consume-token-output]
learned_params: [swiglu-fast-weights-per-block]
failure_modes: [trains-new-weights-vs-training-free-alternatives, muon-optimization-hyperparameter-tuning]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, ttt, fast-weights, vggt-successor]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

SwiGLU fast weights updated via a single Muon-optimized gradient step per layer; queries applied to frozen updated weights as a cross-attention analog. 24-block architecture interleaves local window attention + TTT layer. Fast weights additionally serve as an implicit scene representation queryable with novel ray tokens for feed-forward NVS. Streaming extension: sequential TTT updates serve the same architecture online.

## Why it wins

O(N) linear scaling, 700 frames in <10s on H100, 20× faster than VGGT. Matches VGGT's quadratic accuracy on DTU / ETH3D. Fast weights as scene representation opens real-time queryable NVS.

## Preconditions & compatibility

Trains new weights — the trade vs. TTT3R (training-free) and LoGeR (hybrid with SWA). ZipMap + TTT3R + LoGeR form the 2025–2026 TTT family in [[feed-forward-structure-from-motion]].

## Open questions

- Can the fast-weights-as-scene-representation idea compose with 3DGS fillers, or do the two representations conflict?
