---
title: Adaptive tetrahedral-mesh Gaussian initialization
type: idea
source_paper: wiki/papers/guo2025_ea-3dgs.md
also_in: []

scope: drop-in
stages: [radiance-fields.initialization]
inputs: [colmap-sparse-points, scene-bbox]
outputs: [initial-gaussian-positions-covering-textureless-regions]
assumptions: [sparse-points-adequate-seed, delaunay-feasible-at-point-count]
requires_upstream_property: [colmap-sparse-reconstruction-available]
requires_downstream_property: [3dgs-training-accepts-pre-seeded-positions]
learned_params: []
failure_modes: [tetra-preprocessing-dominates-for-very-large-point-clouds]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, initialization, tetrahedralization, city-scale]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Delaunay tetrahedralize the COLMAP sparse points. On each triangular face of the tetrahedralization, spawn a Gaussian with local-geometry-refined scale/rotation. Coverage in textureless regions (roads, rooftops, terrain) — where sparse SfM points are scarce — is the primary win.

## Why it wins

COLMAP-sparse-point init leaves textureless regions under-densified; the Gaussian densification heuristics then fight to populate those regions from scratch, wasting iterations. Delaunay pre-seeding gives the densification a better starting point. Paper's ablation on aerial scenes shows cleaner coverage over roads/rooftops.

## Open questions

- Can curvature-aware (structure-aware) densification (N3 in the paper) replace the fixed Delaunay seeding?
