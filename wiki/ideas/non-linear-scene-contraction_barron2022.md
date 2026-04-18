---
title: Piecewise non-linear scene contraction for unbounded scenes
type: idea
source_paper: wiki/papers/barron2022_mip-nerf-360.md
also_in: []

scope: drop-in
stages: [radiance-fields.scene-parameterization]
inputs: [unbounded-3d-scene]
outputs: [contracted-coordinates-in-unit-ball]
assumptions: [unbounded-scene, posed-input]
requires_upstream_property: []
requires_downstream_property: [renderer-accepts-contracted-coordinates]
learned_params: []
failure_modes: [far-content-at-disparity-scale-loses-angular-resolution]

requires: []
unlocks: [proposal-network-online-distillation_barron2022, distortion-regularizer_barron2022]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, unbounded-scene, contraction]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Map ℝ³ → ball of radius 2 via a piecewise non-linear function: inside radius 1, identity; outside, compressed so distant content sits proportional to disparity rather than depth. Every radiance-field paper since that handles unbounded captures uses this or a close variant.

## Why it wins

Enables unbounded 360° reconstruction *at all* — finite grids/samplers can't represent infinite volumes. Inherited unchanged by Zip-NeRF and every downstream unbounded-scene method.

## Open questions

- Is disparity-proportional the right far-field parameterization for *outdoor* scenes with complex topology, or would a learned one help?
