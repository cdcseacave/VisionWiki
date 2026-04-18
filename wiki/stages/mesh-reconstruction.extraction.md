---
title: Mesh-reconstruction extraction
type: stage
slug: mesh-reconstruction.extraction
consumes: [opacity-field-or-sdf-field-or-tsdf-voxels]
produces: [triangle-mesh]
invariants: [watertight-if-field-is-consistent]
provides_properties: [standard-dcc-tool-compatibility]
requires_upstream_properties: [level-set-field-available]
data_regime: [post-training-offline]
tags: [marching-cubes, delaunay, tsdf, mesh-extraction]
created: 2026-04-18
updated: 2026-04-18
---

Extracts a triangle mesh from a continuous or discrete level-set field. Marching Cubes is the classical filler; Delaunay-in-loop (MILo) is the training-time variant. Example fillers: [[delaunay-mesh-in-loop_guedon2025]], [[sorted-opacity-hierarchical-resort_radl2025]].
