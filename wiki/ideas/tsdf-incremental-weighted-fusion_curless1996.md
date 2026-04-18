---
title: Incremental weighted-average signed-distance fusion (TSDF)
type: idea
source_paper: wiki/papers/curless1996_tsdf.md
also_in: []

scope: stage-swap
stages: [mesh-reconstruction.depth-fusion]
inputs: [per-frame-depth-map, per-frame-camera-pose, optional-per-pixel-confidence]
outputs: [per-voxel-signed-distance, per-voxel-weight]
assumptions: [static-scene, per-voxel-running-average-OK, sensor-noise-reasonable]
requires_upstream_property: [depth-maps-available-with-poses]
requires_downstream_property: [marching-cubes-or-equivalent-level-set-extractor]
learned_params: []
failure_modes: [surface-drift-near-weight-saturation, no-view-invariant-feature-disentanglement]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [tsdf, mesh-reconstruction, classical, foundational]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Each voxel maintains `(weighted_sum, weight_sum)`. Each new range image projects its depths into the voxel grid; signed distance at each voxel is updated as `weighted_sum += w · d; weight_sum += w`, with `w` a per-sample confidence (from incidence angle / range). Surface extraction (marching cubes) reads the normalized `weighted_sum / weight_sum`. A truncation band limits storage to a narrow zone around the zero level set.

## Why it wins

This is the bottom of Paradigm A's mesh-extraction stack — every method rendering depths from Gaussians + TSDF + MC (VA-GS, Kim 2025, SOF, MILo as baseline) uses this algorithm. The truncation band enables constant-memory-per-surface-area hierarchical variants (VDB, voxel hashing) powering every large-scale TSDF system.

## Preconditions & compatibility

Synthesis bet: use [[per-gaussian-self-supervised-confidence_radl2026]] as the TSDF fusion weight rather than default uniform weighting. Both mechanisms exist; no paper composes them. Candidate component in [[gaussian-to-mesh-pipelines]].

## Open questions

- Is there a learned replacement that maintains the constant-memory property?
