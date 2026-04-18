---
title: Test-time-trained MLP replacing global KV attention (VGG-T3)
type: idea
source_paper: wiki/papers/elflein2026_vgg-t3.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.global-attention]
inputs: [multi-view-tokens, frozen-pretrained-vggt-weights]
outputs: [linear-cost-global-attention-output, meshable-scene-representation]
assumptions: [pretrained-vggt-available, post-training-linearization-OK]
requires_upstream_property: [vggt-style-backbone-pretrained]
requires_downstream_property: [downstream-uses-global-attention-output]
learned_params: [fine-tuned-attention-replacement-weights, l2-normalization-replaces-layernorm]
failure_modes: [inherits-vggt-ceiling, l2-normalization-replacement-fragility]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, ttt, linear-attention, post-training, vggt-linearization]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

At each global-attention layer, Q/K/V projected; KV compressed into a **fixed-size MLP** via a single gradient-step TTT update; Q's passed through the frozen updated MLP as a cross-attention analog. **Post-training linearization**: start from pretrained VGGT, swap LayerNorm → L2 normalization, fine-tune only the attention replacement. No from-scratch retraining. The frozen updated MLP doubles as a **queryable scene representation** for feed-forward visual localization.

## Why it wins

11.6× speedup on 1K images, matching VGGT quality on DTU/ETH3D; 2–2.5× Chamfer reduction over TTT3R. Post-training linearization is the cheapest path to a quadratic → linear migration — no pre-training compute needed.

## Preconditions & compatibility

VGG-T3 outputs mesh/occupancy directly — bridges feed-forward reconstruction to mesh extraction without TSDF/MC post-pass (Paradigm D in [[gaussian-to-mesh-pipelines]]).

## Open questions

- Does post-training linearization work on other pretrained transformers (RoMa v2, MASt3R), or is VGGT's architecture special?
