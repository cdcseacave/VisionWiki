---
title: Distortion regularizer on per-ray weight distribution
type: idea
source_paper: wiki/papers/barron2022_mip-nerf-360.md
also_in: []

scope: drop-in
stages: [radiance-fields.regularization]
inputs: [per-ray-weight-distribution]
outputs: [distortion-loss-term]
assumptions: [volume-rendering-produces-per-ray-weights]
requires_upstream_property: [weight-distribution-differentiable]
requires_downstream_property: [main-photometric-loss-still-dominant]
learned_params: []
failure_modes: [over-regularizes-thin-translucent-surfaces]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, 3dgs, regularization, floater-suppression]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

First-principles penalty on the spread-out-ness of per-ray weights: `L_dist = Σ_{i,j} w_i w_j |t_i - t_j| + Σ_i w_i² (t_{i+1} - t_i) / 3`. Discourages rays whose weight mass is smeared across many samples (the signature of floaters) and encourages concentration at a single depth.

## Why it wins

Rare case of a principled ambiguity fix — most floater suppression is engineering taste. Mechanism is **representation-agnostic**: acts on per-ray weights, which both NeRF and 3DGS produce. Not yet ported to 3DGS/SVRaster — one of the cleanest "port this" bets across [[radiance-field-evolution]].

## Open questions

- Why has no 3DGS paper replicated this despite the mechanism being representation-agnostic?
