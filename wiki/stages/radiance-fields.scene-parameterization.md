---
title: Radiance-fields scene parameterization
type: stage
slug: radiance-fields.scene-parameterization
consumes: [scene-bounds, world-coordinate-cameras]
produces: [parameterized-coordinates]
invariants: [distant-content-representable]
provides_properties: [unbounded-compatibility, sampling-resolution-graceful]
requires_upstream_properties: []
data_regime: [unbounded-scene, 360-degree-captures]
tags: [nerf, contraction, unbounded-scene]
created: 2026-04-18
updated: 2026-04-18
---

Maps world-space to a bounded coordinate system amenable to grids / MLPs. Mip-NeRF 360's piecewise non-linear contraction is the canonical filler. Inherited by every unbounded-scene method since. Example fillers: [[non-linear-scene-contraction_barron2022]].
