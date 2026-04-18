---
title: Feed-forward SfM frame matching
type: stage
slug: feed-forward-sfm.frame-matching
consumes: [frame-pair-pointmaps]
produces: [per-pixel-correspondence]
invariants: [real-time-budget-respected]
provides_properties: [cuda-optimized-matching]
requires_upstream_properties: [mast3r-style-pointmap-output]
data_regime: [online-slam]
tags: [mast3r-slam, pointmap-matching, cuda]
created: 2026-04-18
updated: 2026-04-18
---

Frame-to-frame pointmap projection matching (MASt3R-SLAM iterative variant at 2ms/pixel, replacing the decoder match at 2s/pair). Example fillers: [[iterative-ray-pointmap-matching_murai2025]].
