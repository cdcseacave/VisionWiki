---
title: Per-voxel uncertainty-gated external depth constraint (GeoSVR)
type: idea
source_paper: wiki/papers/li2025_geosvr.md
also_in: []

scope: stage-swap
stages: [mesh-reconstruction.depth-fusion]
inputs: [sparse-voxel-grid, external-depth-priors-possibly-noisy]
outputs: [confidence-gated-depth-supervision]
assumptions: [voxel-representation, external-depth-prior-available, commercial-license-blocked-svraster-inherit]
requires_upstream_property: [per-voxel-uncertainty-estimable]
requires_downstream_property: [loss-accepts-gated-supervision]
learned_params: [per-voxel-uncertainty]
failure_modes: [multi-view-regularization-slow-in-current-impl, uncertainty-calibration-not-validated]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sparse-voxel, uncertainty, depth-supervision, geosvr]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Per-voxel uncertainty values gate external depth supervision: when uncertainty is high (the voxel is not confident about its depth), the external depth prior is accepted as supervision; when uncertainty is low (voxel is already confident), external depth is ignored to avoid biasing. A sparse-voxel surface regularizer encourages sharp surfaces + removes redundant voxels via global geometry consistency across the voxel grid.

## Why it wins

SOTA DTU Chamfer — the strongest purely-voxel-based result. Uncertainty gating is the voxel analog of [[per-gaussian-self-supervised-confidence_radl2026]] — the mechanism is the same (per-primitive scalar dynamically re-weights supervision) but the primitive is a voxel instead of a Gaussian. Bet #006 is the explicit cross-representation composition.

## Preconditions & compatibility

Commercial license blocked (NVIDIA NSCL inherited from SVRaster). Multi-view regularization is slow in the current implementation (acknowledged future-work item).

## Open questions

- Scalability to very large unbounded outdoor scenes not validated.
- Does the uncertainty calibrate to empirical error (i.e. is "high uncertainty" actually correlated with high depth error)?
