---
title: Flattened-Gaussian constraint + unbiased depth rendering (PGSR)
type: idea
source_paper: wiki/papers/chen2024_pgsr.md
also_in: []

scope: stage-swap
stages: [radiance-fields.regularization]
inputs: [3d-gaussians, per-pixel-view-direction]
outputs: [planar-gaussians, unbiased-depth-map]
assumptions: [static-scene, surface-adherent-scene-default]
requires_upstream_property: [gaussian-positions-covariances-trainable]
requires_downstream_property: [renderer-accepts-depth-via-plane-intersection]
learned_params: []
failure_modes: [genuinely-thin-non-planar-structure-resists-flattening]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, planar-constraint, unbiased-depth, regularization]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Smallest-eigenvalue penalty on the 3D covariance drives Gaussians toward oriented disks (planar primitives). Depth = `(distance-from-camera-to-plane) / (view-direction · plane-normal)` — an unbiased depth that replaces α-composited depth. Single-view + multi-view geometric consistency losses (PatchMatch-style) regularize the primitives without external priors.

## Why it wins

Closes most of the gap to neural-SDF methods while keeping 3DGS's speed. SOTA on DTU/T&T/Mip-360 among 3DGS surface methods. Depth is first-class: α-composited depth was silently biased and no prior paper caught this. Works without external MVS priors — a strong argument that "3DGS + surfaces" is a *regularization* problem, not a representation problem.

## Preconditions & compatibility

Multi-view term is training-expensive (cross-view photometric re-projection per iteration). Hair/foliage/thin structures resist planar flattening. ZJU research-only license (non-commercial).

## Open questions

- Can the planar constraint and unbiased depth be decoupled? The paper ships them together but they're conceptually separable.
