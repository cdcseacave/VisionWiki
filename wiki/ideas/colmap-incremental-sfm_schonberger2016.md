---
title: COLMAP incremental SfM (multi-model RANSAC + next-best-view + iterative triangulation+BA)
type: idea
source_paper: wiki/papers/schonberger2016_colmap-sfm.md
also_in: []

scope: new-paradigm
stages: [sfm.geometric-verification, sfm.next-view-scheduling, sfm.iterative-triangulation-ba]
collapses: []
splits_into: []
rewrites: {}

inputs: [unordered-images, feature-matches]
outputs: [posed-cameras, sparse-3d-points, coordinate-frame]
assumptions: [static-scene, sufficient-overlap, cpu-acceptable]
requires_upstream_property: [feature-matches-available]
requires_downstream_property: [downstream-pipelines-accept-sparse-point-cloud-output]
learned_params: []
failure_modes: [cpu-bound-bottleneck-on-ba, panorama-planar-degeneracies-caught-by-multi-model]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, colmap, classical, foundational, accuracy-baseline]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Multi-model geometric verification**: RANSAC over homography + epipolar models catches panorama/planar degeneracies. **Expected-gain next-best-view selection**: picks next camera by predicted reconstruction gain, not match count. **Iterative triangulation + BA loop**: re-triangulates as BA refines poses, recovering tracks lost to initial noise.

## Why it wins

The **accuracy reference every faster SfM paper measures against**. Ground-truth pose source for [[radiance-field-evolution]], [[gaussian-to-mesh-pipelines]], most 3DGS benchmarks — COLMAP poses define the coordinate frame of every published reconstruction number.

## Preconditions & compatibility

Current SOTA in [[gpu-native-sfm]] is not a swap for COLMAP's components but a *reimplementation* of the same math on GPU (InstantSfM, CuSfM). The swap axis is implementation, not algorithm — which is the core thesis of that thread.

## Open questions

- Is there a GPU implementation that preserves COLMAP's accuracy while also matching its robustness on pathological internet collections?
