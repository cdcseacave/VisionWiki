---
title: Per-Gaussian discretized SDF samples + SDF-to-opacity bridge
type: idea
source_paper: wiki/papers/zhu2025_gs-discretized-sdf.md
also_in: []

scope: stage-swap
stages: [radiance-fields.primitive-representation, mesh-reconstruction.extraction, relighting.inverse-rendering]
collapses: []
splits_into: []
rewrites: {}

inputs: [3d-gaussians, per-gaussian-scalar-sdf-sample]
outputs: [sdf-aware-gaussian-set, relightable-mesh]
assumptions: [static-scene, posed-input]
requires_upstream_property: [3dgs-training-pipeline]
requires_downstream_property: [renderer-accepts-sdf-opacity-mapping]
learned_params: [gaussian.sdf-value, sh-coeffs]
failure_modes: [discrete-sampling-misses-sub-gaussian-detail]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, sdf, relighting, mesh-extraction]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Each Gaussian stores a scalar SDF sample (no separate continuous SDF network). A differentiable SDF-to-opacity mapping links the discrete SDF to the Gaussian's opacity; a **median loss** pulls Gaussians onto the zero level set. A projection-based consistency loss (discrete-SDF analog of Eikonal) enforces alignment between splat surface and SDF surface.

## Why it wins

Half the memory of dual SDF + Gaussian networks (GS2Mesh, 3DGSR), simpler training loop, SOTA relighting on Glossy Blender. Unifies mesh extraction + relighting + appearance in one representation.

## Preconditions & compatibility

Adds the **relightable / inverse-rendering lane** to [[gaussian-to-mesh-pipelines]] — no pure-mesh-extraction paper addresses relighting. Synthesis bet: CoMe's self-supervised confidence as complementary signal to the SDF confidence.

## Open questions

- Discrete sampling misses detail smaller than Gaussian extent.
- Does the median loss compose with CoMe's confidence loss without interference?
