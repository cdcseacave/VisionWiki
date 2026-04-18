---
title: DUSt3R-MVS + voxel-guided sparse-view 3DGS regularization
type: idea
source_paper: wiki/papers/tang2025_dronesplat.md
also_in: []

scope: stage-swap
stages: [radiance-fields.initialization, radiance-fields.sparse-view-regularization]
inputs: [sparse-views, dust3r-mvs-output]
outputs: [voxel-guided-gaussian-init, depth-distortion-loss]
assumptions: [sparse-view-input, dust3r-available-for-init]
requires_upstream_property: [feed-forward-mvs-produces-dense-points]
requires_downstream_property: [3dgs-training-accepts-voxel-grid-regularizer]
learned_params: []
failure_modes: [mvs-quality-bounds-init, voxel-resolution-tradeoff]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, sparse-view, dust3r, voxel-init]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Dense 3D points from DUSt3R-based MVS replace COLMAP sparse points as the Gaussian initialization. A voxel grid built from those points guides 3DGS training via depth-distortion + geometric constraints — when the voxel says "no surface here," the Gaussians get regularized away from that region.

## Why it wins

+2 dB over next-best sparse-view baseline on 6-view Simingshan. Supersedes earlier InstantSplat-style init with DUSt3R-specific voxel guidance. The combination of dense MVS init + voxel regularizer is load-bearing — either alone is weaker.

## Open questions

- DUSt3R inference overhead is non-trivial; is there a cheaper MVS option that preserves the init quality?
