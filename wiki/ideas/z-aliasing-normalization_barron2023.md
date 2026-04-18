---
title: Z-aliasing normalization for proposal weights
type: idea
source_paper: wiki/papers/barron2023_zip-nerf.md
also_in: []

scope: drop-in
stages: [radiance-fields.importance-sampling]
inputs: [raw-proposal-weights-across-scales]
outputs: [normalized-proposal-weights]
assumptions: [grid-based-proposal-sampling, scale-variance-observable]
requires_upstream_property: [proposal-weights-differentiable]
requires_downstream_property: [downstream-sampling-uses-normalized-weights]
learned_params: []
failure_modes: [empirical-no-principled-explanation]

requires: [proposal-network-online-distillation_barron2022]
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, proposal-network, aliasing, empirical]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Normalize proposal weights across scales before downstream sampling. Suppresses along-ray jitter that appears in grid-based proposal sampling (the Z-aliasing artifact Zip-NeRF identified).

## Why it wins

Empirical fix — the paper shows the artifact clearly in ablations, but offers no principled explanation for why normalization eliminates it. Remains an open theoretical question noted in [[radiance-field-evolution]].

## Open questions

- Why do normalized weights eliminate the artifact? No paper has closed this.
