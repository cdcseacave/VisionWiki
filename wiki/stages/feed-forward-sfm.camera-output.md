---
title: Feed-forward SfM camera output
type: stage
slug: feed-forward-sfm.camera-output
consumes: [dense-per-pixel-prediction]
produces: [invertible-camera-extrinsics-intrinsics]
invariants: [camera-recoverable-from-output]
provides_properties: [unified-with-geometry-output]
requires_upstream_properties: [dense-prediction-head]
data_regime: [diffusion-sfm]
tags: [diffusion-sfm, camera-output]
created: 2026-04-18
updated: 2026-04-18
---

Unified per-pixel output where ray origins invert to cameras (DiffusionSfM). Not separable from [[feed-forward-sfm.geometry-output]] — the two share fillers. Example fillers: [[ray-origin-endpoint-diffusion_zhao2025]].
