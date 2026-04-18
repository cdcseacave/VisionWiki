---
title: Feed-forward SfM global optimization
type: stage
slug: feed-forward-sfm.global-optimization
consumes: [keyframe-pose-graph, per-frame-pointmaps]
produces: [globally-consistent-poses]
invariants: [sparse-cholesky-solvable]
provides_properties: [realtime-or-online-global-opt]
requires_upstream_properties: [per-frame-pose-graph-available]
data_regime: [slam]
tags: [mast3r-slam, sim3-opt, gauss-newton]
created: 2026-04-18
updated: 2026-04-18
---

Gauss-Newton Sim(3) global optimization with analytic CUDA Jacobians (MASt3R-SLAM). 15 FPS end-to-end on MASt3R priors. Example fillers: [[iterative-ray-pointmap-matching_murai2025]].
