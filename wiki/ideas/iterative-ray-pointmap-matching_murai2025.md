---
title: Iterative pointmap projection matching + ray-error pose + Sim(3) global opt (MASt3R-SLAM)
type: idea
source_paper: wiki/papers/murai2025_mast3r-slam.md
also_in: []

scope: topology-rewrite
stages: [feed-forward-sfm.frame-matching, feed-forward-sfm.pose-estimation, feed-forward-sfm.temporal-fusion, feed-forward-sfm.global-optimization]
collapses: []
splits_into: []
rewrites: {replaces: [feed-forward-sfm.frame-matching, feed-forward-sfm.pose-estimation], introduces: [feed-forward-sfm.ray-based-matching, feed-forward-sfm.ray-error-pose, feed-forward-sfm.running-pointmap-fusion, feed-forward-sfm.sim3-global-opt]}

inputs: [mast3r-pointmap-predictions, per-frame-ray-directions]
outputs: [realtime-slam-output, calibration-free-poses]
assumptions: [mast3r-available, generic-camera-model-ok, 2ms-per-frame-matching-budget]
requires_upstream_property: [mast3r-inference-available]
requires_downstream_property: [consumer-accepts-sim3-poses]
learned_params: []
failure_modes: [mast3r-depth-prediction-errors-propagate]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, slam, mast3r, ray-based, realtime]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Iterative pointmap projection matching**: converges 10 iterations/pixel in 2ms via custom CUDA — replaces MASt3R's feature-decoder match (2s/pair). **Directional ray-error pose**: Sim(3) pose minimizing ray-direction residual (not 3D-point residual) — robust to MASt3R depth-prediction errors. **Running weighted-average pointmap fusion**: online filter over per-frame predictions. **Gauss-Newton Sim(3) global opt**: sparse Cholesky over pose graph with analytic CUDA Jacobians.

## Why it wins

First real-time feed-forward SLAM on DUSt3R/MASt3R priors: 15 FPS end-to-end, no calibration needed. The four mechanisms address four bottlenecks: matching speed (1000× over MASt3R decoder), depth-error robustness, temporal noise, global consistency.

## Pipeline-shape implications

Topology-rewrite: replaces MASt3R-SfM's matching + pose stages with a four-stage online pipeline. Any thread adopting MASt3R-SLAM uses this expanded DAG.
