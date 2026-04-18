---
title: Feed-forward SfM temporal fusion
type: stage
slug: feed-forward-sfm.temporal-fusion
consumes: [per-frame-pointmap-predictions, temporal-order]
produces: [time-averaged-consistent-pointmap]
invariants: [noise-averages-out, drift-bounded]
provides_properties: [consistent-camera-model-over-time]
requires_upstream_properties: [per-frame-predictions-in-sequence]
data_regime: [online-slam, video]
tags: [mast3r-slam, pointmap-fusion, running-average]
created: 2026-04-18
updated: 2026-04-18
---

Running weighted-average fusion over per-frame pointmap predictions. Denoises MASt3R's per-frame errors; classical EKF-SLAM filtering pattern applied to learned predictions. Example fillers: [[iterative-ray-pointmap-matching_murai2025]].
