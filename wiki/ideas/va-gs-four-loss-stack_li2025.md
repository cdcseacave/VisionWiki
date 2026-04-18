---
title: VA-GS four-loss stack (visibility + edge + normal + deep-feature alignment)
type: idea
source_paper: wiki/papers/li2025_va-gs.md
also_in: []

scope: stage-swap
stages: [radiance-fields.regularization]
inputs: [3d-gaussians, multi-view-images, per-pixel-features]
outputs: [regularization-loss-terms-set]
assumptions: [static-scene, moderate-multi-view-overlap, no-license-repo-no-LICENSE-file]
requires_upstream_property: [multi-view-rendering-available, per-view-feature-extractor]
requires_downstream_property: [tolerates-four-simultaneous-losses]
learned_params: []
failure_modes: [feature-alignment-limited-on-diverse-lighting-benchmarks]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, multi-loss, regularization, feature-alignment]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Four simultaneous loss terms: (a) **visibility-aware photometric alignment** `L_p` — reprojection-error-weighted multi-view photometric loss, downweighting occluded/misaligned pixels; (b) **edge-aware image loss** — image-edge cues in the rendering loss; (c) **normal orientation + smoothing** `L_nc/L_ns` — refines Gaussian spatial orientation; (d) **multi-view deep-feature alignment** `L_f` — cross-view feature consistency via deep embeddings.

## Why it wins

SOTA DTU Chamfer + T&T F1 among 3DGS surface methods (as of 2025). The visibility-aware term specifically removes shadow/specular artifacts that simpler photometric losses absorb as floaters.

## Preconditions & compatibility

Feature alignment (`L_f`) shows limited benefit on T&T (diverse lighting) vs. DTU (clean conditions) — swapping generic features for DINOv3 is a natural bet (noted in paper's Pipeline contribution as N4 synthesis-bet candidate). **No LICENSE file in repo** → treat as all-rights-reserved; blocks redistribution without explicit author permission.

## Open questions

- Which of the four losses is individually load-bearing? Paper ablates additively, not individually.
- Does Bet #005 (PGSR + VA-GS + CoMe + MVS-depth stacked) suffer loss-term interference?
