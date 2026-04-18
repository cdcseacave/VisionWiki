---
title: Radiance-fields grid-feature sampling
type: stage
slug: radiance-fields.grid-feature-sampling
consumes: [hash-grid-or-voxel-grid, per-ray-cone-or-point]
produces: [feature-at-sample-location]
invariants: [multisampling-matches-cone-footprint]
provides_properties: [anti-aliased-at-scale-changes]
requires_upstream_properties: [grid-representation-available]
data_regime: [grid-based-renderers]
tags: [zip-nerf, instant-ngp, hash-grid, anti-aliasing]
created: 2026-04-18
updated: 2026-04-18
---

Queries a hash/voxel grid at per-ray locations. Multisample anti-aliased query (Zip-NeRF) approximates a cone with isotropic-Gaussian samples, feature-averaged. Example fillers: [[multisample-anti-aliased-hash-grid_barron2023]].
