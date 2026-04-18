---
title: Median-depth-based relative-depth loss with learned uncertainty (external MVS)
type: idea
source_paper: wiki/papers/kim2025_multiview-geometric-gs.md
also_in: []

scope: stage-swap
stages: [radiance-fields.depth-supervision]
inputs: [3d-gaussians, external-mvs-depth-possibly-noisy, rendered-depth]
outputs: [median-depth-relative-loss, per-pixel-uncertainty-weighted-supervision]
assumptions: [external-mvs-available, mvs-quality-bounded]
requires_upstream_property: [mvs-depth-pipeline-output]
requires_downstream_property: [loss-accepts-uncertainty-weighted-residual]
learned_params: [per-pixel-uncertainty]
failure_modes: [mvs-depth-overhead-4-35-min, uncertainty-model-per-pixel-not-principled]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: [per-gaussian-self-supervised-confidence_radl2026]

tags: [3dgs, mvs, depth-supervision, external-prior]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Use **median** rather than mean depth for the rendered-depth estimator — α-composited mean is biased by semi-transparent Gaussian ambiguity. Compare depth *ratios* between neighboring pixels (relative depth) rather than absolute depth against MVS. A learned per-pixel uncertainty downweights unreliable MVS regions.

## Why it wins

Establishes "external MVS > Gaussian self-supervision for mesh quality" on DTU/T&T — the signature finding of Paradigm A. Relative-depth comparison neutralizes MVS scale noise; median depth handles the bias.

## Preconditions & compatibility

**Contradicts** [[per-gaussian-self-supervised-confidence_radl2026]] (CoMe): CoMe argues self-supervised confidence wins, Kim 2025 argues external MVS wins. Resolution hypothesis in thread: external wins on controlled capture, self-supervised wins on sparse / textureless. MVS adds 4–35 min preprocessing.

## Open questions

- Highly reflective / transparent surfaces where MVS and 3DGS both struggle not evaluated.
- Principled geometric uncertainty model would beat per-pixel learned uncertainty.
