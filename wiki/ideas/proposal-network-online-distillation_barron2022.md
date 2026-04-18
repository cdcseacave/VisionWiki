---
title: Proposal-network online distillation for importance sampling
type: idea
source_paper: wiki/papers/barron2022_mip-nerf-360.md
also_in: []

scope: stage-swap
stages: [radiance-fields.importance-sampling]
inputs: [ray-samples, nerf-mlp-weights]
outputs: [sampled-points-concentrated-near-surface]
assumptions: [two-mlp-setup, nerf-training-budget]
requires_upstream_property: [base-mlp-available]
requires_downstream_property: [renderer-consumes-proposed-samples]
learned_params: [proposal-mlp-weights]
failure_modes: [proposal-mismatch-at-very-thin-surfaces]

requires: [non-linear-scene-contraction_barron2022]
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, importance-sampling, proposal-network, distillation]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Small proposal MLP distilled from the large NeRF MLP's weight histogram via an upper-envelope-matching loss. The proposal network is cheap; it predicts where along each ray to sample densely. Replaces NeRF's coarse/fine two-MLP setup with a distillation-trained proposal + one large NeRF MLP.

## Why it wins

Higher-capacity NeRF at moderate cost; the large MLP's forward passes are concentrated where they matter. Carried forward unchanged into Zip-NeRF — remains the canonical NeRF-family importance-sampler.

## Open questions

- Can the same distillation pattern be applied to 3DGS densification (a "proposal MLP for which Gaussians to split")?
