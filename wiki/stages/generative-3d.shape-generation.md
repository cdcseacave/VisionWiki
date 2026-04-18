---
title: Generative-3d shape generation
type: stage
slug: generative-3d.shape-generation
consumes: [single-or-few-images, optional-per-object-masks]
produces: [per-object-geometry-and-texture]
invariants: [per-object-output-separable]
provides_properties: [generalizes-from-synthetic-to-real]
requires_upstream_properties: [shape-generator-pretrained]
data_regime: [generative-from-underconstrained-input]
tags: [sam-3d, flow-matching, single-image]
created: 2026-04-18
updated: 2026-04-18
---

Produces per-object geometry + texture from underconstrained input (single image). Latent flow-matching is the SAM 3D filler. Example fillers: [[two-stage-latent-flow-matching-scene_chen2025]].
