---
title: Feed-forward SfM pose estimation
type: stage
slug: feed-forward-sfm.pose-estimation
consumes: [pointmap-predictions, ray-directions]
produces: [sim3-or-se3-poses]
invariants: [depth-error-robustness]
provides_properties: [closed-form-or-iterative-pose-recovery]
requires_upstream_properties: [pointmap-or-pointcloud-predictions]
data_regime: [feed-forward-reconstruction]
tags: [mast3r-slam, ray-error, procrustes]
created: 2026-04-18
updated: 2026-04-18
---

Recovers poses from feed-forward pointmap outputs. Ray-direction error (MASt3R-SLAM) is robust to depth-scale errors; Procrustes from a third pointmap (Pow3R) gives single-pass two-camera recovery. Example fillers: [[iterative-ray-pointmap-matching_murai2025]], [[third-pointmap-procrustes_jang2025]].
