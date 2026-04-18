---
title: Radiance-fields primitives
type: stage
slug: radiance-fields.primitives
consumes: [scene-bounds, initialization-points]
produces: [primitive-set-gaussians-or-voxels-or-implicit]
invariants: [primitive-type-fixed-per-training-run]
provides_properties: [rasterizable, per-primitive-attributes-learnable]
requires_upstream_properties: [scene-initialization-available]
data_regime: [static-scene, bounded-or-unbounded]
tags: [3dgs, svraster, primitive]
created: 2026-04-18
updated: 2026-04-18
---

The choice of underlying primitive: 3D Gaussians (3DGS), sparse voxels (SVRaster), or implicit MLP+SDF (AniSDF). Fillers define what a primitive *is*; composition with downstream rendering stages is type-dependent. Example fillers: [[morton-ordered-sparse-voxel-rasterizer_sun2025]], [[per-gaussian-discretized-sdf_zhu2025]].
