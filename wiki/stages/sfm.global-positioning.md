---
title: SfM global positioning
type: stage
slug: sfm.global-positioning
consumes: [pairwise-relative-poses, view-graph]
produces: [global-camera-centers, sparse-3d-points]
invariants: [metric-scale-preserved-if-depth-priors-available]
provides_properties: [initialization-for-ba, global-coordinate-frame]
requires_upstream_properties: [geometric-verification-output]
data_regime: [static-scene]
tags: [sfm, global-sfm, glomap, instantsfm]
created: 2026-04-18
updated: 2026-04-18
---

Joint global solve for cameras + points (GLOMAP-style) or depth-constrained version (InstantSfM). Replaces the older translation-averaging → BA two-stage. Example fillers: [[glomap-joint-global-positioning_pan2024]], [[depth-constrained-jacobian_zhong2026]].
