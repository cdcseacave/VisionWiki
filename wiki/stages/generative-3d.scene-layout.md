---
title: Generative-3d scene layout
type: stage
slug: generative-3d.scene-layout
consumes: [per-object-generations, image-context]
produces: [per-object-placement-poses-scales]
invariants: [objects-don't-overlap-physically-infeasibly]
provides_properties: [composable-scene-output]
requires_upstream_properties: [shape-generation-done]
data_regime: [generative]
tags: [sam-3d, layout, composition]
created: 2026-04-18
updated: 2026-04-18
---

Predicts per-object placement into the scene. Example fillers: [[two-stage-latent-flow-matching-scene_chen2025]].
