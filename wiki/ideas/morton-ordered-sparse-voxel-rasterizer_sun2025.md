---
title: Ray-direction Morton-ordered sparse-voxel rasterizer
type: idea
source_paper: wiki/papers/sun2025_sparse-voxels-rasterization.md
also_in: []

scope: new-paradigm
stages: [radiance-fields.primitives, radiance-fields.rendering]
collapses: []
splits_into: []
rewrites: {replaces: [radiance-fields.primitives, radiance-fields.rendering], introduces: [radiance-fields.sparse-voxel-primitives, radiance-fields.morton-rasterization]}

inputs: [posed-images, sparse-voxel-grid, per-voxel-density-sh]
outputs: [rendered-image, correctly-depth-ordered-volume-integration]
assumptions: [static-scene, posed-input, grid-representation-compatible, commercial-license-blocked-NSCL]
requires_upstream_property: [adaptive-voxel-allocation-tunable]
requires_downstream_property: [consumer-is-agnostic-to-primitive-type-or-voxel-native]
learned_params: [per-voxel-density, per-voxel-sh-coeffs]
failure_modes: [48bit-morton-sort-cost-vs-3dgs-32bit-float-sort, scene-dependent-fps-variance]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [radiance-field, sparse-voxel, rasterization, morton-sort, neural-free]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Adaptive sparse voxels are Morton-coded (48-bit) in ray direction for each pixel — this gives *correct* per-ray front-to-back depth ordering without ray casting. Tile-based rasterization (as in 3DGS) provides the speed. Post-activation density (density applied after interpolation, not before) gives sharp boundaries rather than Gaussian falloff. No neural networks, no Gaussians — a pure grid-based primitive with native mesh extractability via [[marching-cubes]].

## Why it wins

No popping artifacts (3DGS's depth-sort-by-center is *incorrect* for volume rendering; Morton-ordered sort is correct). 27 primitives/pixel vs. 3DGS's 63 — sharper detail at lower primitive count. Compatible with 30 years of voxel tooling (TSDF, MC, voxel pooling, feature lifting — see LangSVR's Pipeline V).

## Preconditions & compatibility

Requires adaptive voxel allocation tuning per scene. Sort cost is higher than 3DGS's 32-bit float sort (48-bit Morton) — the per-scene FPS variance reflects this trade-off. Commercial license blocked (NVIDIA NSCL).

## Pipeline-shape implications

New-paradigm scope: replaces 3DGS's primitive + rasterization stages with voxels + Morton-rasterization. Threads adopting SVRaster (op:neural-free in radiance-field-evolution, op:natively-extractable in gaussian-to-mesh) use a parallel pipeline, not a stage swap.

## Trade-offs vs. the decomposed pipeline

3DGS pipelines have rich per-primitive editing tooling (select, move, recolor); voxel-grid pipelines lose per-instance addressability unless a parallel identity channel is added. Feature-lifting methods built on Gaussians (LangSplat's autoencoder trick) need re-derivation for voxels — partially addressed by [[wu2026_langsvr]].

## Open questions

- Every synthesis bet of the form "3DGS with X regularization" has a counterpart "SVRaster with X" — only CoMe-style confidence (Bet #001) explicitly tests this.
