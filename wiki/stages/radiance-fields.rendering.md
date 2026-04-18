---
title: Radiance-fields rendering
type: stage
slug: radiance-fields.rendering
consumes: [primitive-set, camera-pose, camera-intrinsics]
produces: [rendered-image, optional-rendered-depth, optional-rendered-feature-channels]
invariants: [gradient-flows-through-render-to-primitives]
provides_properties: [differentiable, batch-tile-based-or-ray-march]
requires_upstream_properties: [primitive-set-with-attributes]
data_regime: [static-scene, posed-input]
tags: [rasterization, volume-rendering, 3dgs, svraster]
created: 2026-04-18
updated: 2026-04-18
---

The forward render from primitives to an image (and auxiliary channels). 3DGS rasterization, SVRaster Morton-ordered rasterization, NeRF volume rendering are all fillers. Feature-channel trick (rendering arbitrary per-primitive vectors by stacking them into the color tensor) lives here. Example fillers: [[morton-ordered-sparse-voxel-rasterizer_sun2025]].
