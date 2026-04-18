---
title: Single-view depth lifting + depth-constrained BA + forward-backward consistency (MP-SfM)
type: idea
source_paper: wiki/papers/pataki2025_mp-sfm.md
also_in: []

scope: topology-rewrite
stages: [sfm.next-view-registration, sfm.bundle-adjustment, sfm.registration-validation]
collapses: []
splits_into: []
rewrites: {replaces: [sfm.next-view-registration, sfm.bundle-adjustment], introduces: [sfm.mono-depth-lifted-registration, sfm.depth-constrained-ba, sfm.forward-backward-depth-consistency]}

inputs: [unordered-images, metric3dv2-depth-and-normal]
outputs: [sfm-reconstruction-on-low-overlap-captures]
assumptions: [mono-depth-prior-available, static-scene]
requires_upstream_property: [metric3dv2-or-equivalent-depth-normal]
requires_downstream_property: [ceres-BA-accepts-depth-residuals]
learned_params: []
failure_modes: [depth-prior-quality-bounds-reconstruction, heterogeneous-gpu-cpu-pipeline]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, mono-depth, depth-normal-prior, low-overlap]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Single-view 3D lifting**: lift per-image features to 3D via Metric3Dv2 depth — enables next-view registration from only two-view tracks (classical incremental SfM needs 3-view). **Depth-constrained BA + normal integration**: alternating depth refinement (normal integration with uncertainty weighting) + joint BA over poses, points, refined depths. **Forward-backward depth consistency**: reprojected-depth agreement identifies symmetry/duplicate-registration failures — resolves ghost-camera problem.

## Why it wins

Works on low-overlap / low-parallax captures where COLMAP fails: AUC@1 34.9 vs GLOMAP 8.4 on ETH3D 0% overlap. Eliminates the 3-view-overlap requirement that makes COLMAP fragile on sparse drone / indoor imagery.

## Preconditions & compatibility

Less accurate than MASt3R-SfM at AUC@1° on object-centric scenes (Tanks & Temples). Heterogeneous compute (GPU depth + CPU Ceres BA). Bet #010 explores fusing MP-SfM's depth+normal priors with InstantSfM's GPU BA.

## Pipeline-shape implications

Topology-rewrite: adds a depth-lifting stage before registration, a forward-backward-consistency stage after registration. A thread adopting this must represent the three-node subgraph, not just a BA swap.
