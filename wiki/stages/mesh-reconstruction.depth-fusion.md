---
title: Mesh-reconstruction depth fusion
type: stage
slug: mesh-reconstruction.depth-fusion
consumes: [per-view-depth-maps, per-view-poses, optional-per-pixel-confidence]
produces: [voxelized-signed-distance-with-weights]
invariants: [constant-memory-per-surface-area-with-truncation-band]
provides_properties: [incremental-update-possible]
requires_upstream_properties: [depth-maps-with-poses]
data_regime: [mesh-pipeline]
tags: [tsdf, curless-levoy, geosvr]
created: 2026-04-18
updated: 2026-04-18
---

Classical TSDF weighted-average fusion or uncertainty-gated voxel supervision. Canonical fillers: [[tsdf-incremental-weighted-fusion_curless1996]], [[voxel-uncertainty-depth-constraint_li2025]].
