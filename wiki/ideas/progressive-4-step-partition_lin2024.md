---
title: Progressive 4-step scene partition for cell-parallel 3DGS training
type: idea
source_paper: wiki/papers/lin2024_vastgaussian.md
also_in: []

scope: stage-swap
stages: [radiance-fields.partitioning]
inputs: [camera-positions, scene-bbox, per-camera-matched-points]
outputs: [per-cell-camera-set, per-cell-point-initialization, post-training-cell-membership]
assumptions: [flat-ish-ground, moderate-camera-density, static-scene]
requires_upstream_property: [sparse-point-cloud-available]
requires_downstream_property: [per-cell-training-independent, merge-on-cell-boundary-OK]
learned_params: []
failure_modes: [storage-scales-linearly-with-cell-count, no-automatic-cell-count-selection]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, city-scale, partitioning, parallel-training]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Four steps: (a) **region-division** — `m × n` ground-plane grid with row-subdivision to balance camera counts; (b) **position-based selection** — expand each cell outward by ~20% and assign cameras inside the expanded rectangle; (c) **visibility-based selection** — add cameras whose airspace visibility ([[airspace-aware-visibility-partitioning_lin2024]]) exceeds `T_h`; (d) **coverage-based point selection** — include all points visible to selected cameras, even outside the cell. Each cell trains a standard 3DGS independently; at merge time, Gaussians outside the original un-expanded cell are discarded — the overlap region during training ensures seamless merge without post-hoc blending.

## Why it wins

Seamless merge without the appearance re-alignment Block-NeRF needs; 10× shorter training vs Mega-NeRF (2h56m vs 29h19m on UrbanScene3D). Overlap during training is the key insight — per-cell Gaussians in the overlap region see identical supervision in both cells, so they agree at the boundary.

## Preconditions & compatibility

Linear storage cost with cell count → layering compression (EA-3DGS) or incremental capture (VPGS-SLAM) is the natural composition (Bet #002). Requires sparse point-cloud + camera positions pre-computed — InstantSfM / COLMAP are the usual upstream.

## Open questions

- Optimal cell count is empirical (8 is best for their aerial scenes); no method to pick from scene geometry.
- Non-planar scenes need a different partitioning primitive.
