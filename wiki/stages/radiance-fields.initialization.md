---
title: Radiance-fields initialization
type: stage
slug: radiance-fields.initialization
consumes: [sparse-points-or-dense-points, scene-bounds]
produces: [initial-primitive-positions, initial-scales-covariances]
invariants: [coverage-matches-input-distribution]
provides_properties: [sufficient-seed-for-densification]
requires_upstream_properties: [sfm-sparse-or-mvs-dense-points]
data_regime: [static-scene, posed-input]
tags: [3dgs, initialization, svraster]
created: 2026-04-18
updated: 2026-04-18
---

Seeds the primitive set before training. COLMAP sparse, MVS dense, or Delaunay tetrahedralization (EA-3DGS). Choice affects downstream densification budget. Example fillers: [[adaptive-tetrahedral-init_guo2025]], [[mvs-voxel-guided-sparse-view-regularization_tang2025]].
