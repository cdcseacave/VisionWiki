---
title: Feed-forward SfM camera parameterization
type: stage
slug: feed-forward-sfm.camera-parameterization
consumes: [patch-or-pixel-features]
produces: [ray-based-or-global-pose-representation]
invariants: [parameterization-invertible-to-extrinsics-intrinsics]
provides_properties: [transformer-friendly-distributed-output]
requires_upstream_properties: [patch-token-features]
data_regime: [transformer-regression]
tags: [plucker-rays, pose-parameterization, raydiffusion]
created: 2026-04-18
updated: 2026-04-18
---

How cameras are represented in the regression head. Plucker ray-bundles (Zhang 2024) or pointmap patches (DUSt3R) vs. global (R, t, K) outputs. Example fillers: [[plucker-ray-bundle-camera_zhang2024]].
