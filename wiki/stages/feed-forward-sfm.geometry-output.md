---
title: Feed-forward SfM geometry output
type: stage
slug: feed-forward-sfm.geometry-output
consumes: [dense-per-pixel-prediction]
produces: [per-pixel-ray-endpoint-or-pointmap]
invariants: [consistent-with-camera-output]
provides_properties: [unified-with-camera-output]
requires_upstream_properties: [dense-prediction-head]
data_regime: [diffusion-sfm]
tags: [diffusion-sfm, pointmap-like]
created: 2026-04-18
updated: 2026-04-18
---

Pixel-wise geometry prediction paired with camera output (DiffusionSfM). Ray-endpoint fields bounded via homogeneous coordinates for stable diffusion. Example fillers: [[ray-origin-endpoint-diffusion_zhao2025]].
