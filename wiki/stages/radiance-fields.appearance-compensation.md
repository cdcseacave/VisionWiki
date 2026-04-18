---
title: Radiance-fields appearance compensation
type: stage
slug: radiance-fields.appearance-compensation
consumes: [rendered-image, gt-image, per-image-metadata]
produces: [compensated-rendered-image, residual-photometric-loss]
invariants: [compensation-vanishes-at-inference, no-per-scene-geometry-change]
provides_properties: [exposure-tonemap-robustness, in-the-wild-capture-tolerance]
requires_upstream_properties: [per-image-embedding-or-metadata-available]
data_regime: [static-scene, posed-input, training-time-only]
tags: [3dgs, appearance-embedding, vastgaussian, ssim]
created: 2026-04-18
updated: 2026-04-18
---

## Description

Absorb training-time appearance drift (exposure, tone-mapping, time-of-day, weather) via a per-image compensation applied to the rendered image before the photometric loss. Gradient flows through the compensation into the appearance embedding, not into the radiance field — at inference the compensation is discarded. Prevents the radiance field from baking transient appearance into its geometry.

## Example fillers

- [[decoupled-appearance-2d-transform_lin2024]] — VastGaussian per-image CNN transformation map.
- [[ssim-decoupled-appearance-embedding_radl2026]] — refinement allowing luminance-only compensation.
- NeRF-W Generative Latent Optimization embedding — implicit-NeRF era; subsumed by the 2D-transform approach.

## Valid-filler notes

The compensation must be training-time-only; a filler that alters the radiance field at inference is a different stage (e.g. NeRF-in-the-wild's appearance latent input). Fillers must not enable the compensation module to mask geometric errors — this is the concrete failure mode the CoMe refinement addresses.
