---
title: SLAM-prior two-stage factor graph + rig extrinsic refinement (CuSfM)
type: idea
source_paper: wiki/papers/yu2025_cusfm.md
also_in: []

scope: stage-swap
stages: [sfm.view-graph-construction, sfm.optimization-schedule, sfm.rig-extrinsic-refinement]
inputs: [image-sequence, slam-prior-trajectory, rig-camera-configuration]
outputs: [refined-poses, refined-rig-extrinsics]
assumptions: [sequential-data, prior-slam-trajectory-available, rigid-rig]
requires_upstream_property: [pycuvslam-or-orb-slam2-output]
requires_downstream_property: [ceres-solver-available-for-BA]
learned_params: [aliked-feature-weights]
failure_modes: [cannot-operate-from-scratch, time-varying-calibration-unsupported]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, factor-graph, slam-prior, multi-camera-rig, tensorrt]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Non-redundant data association**: build a minimal view graph via radius search + sequential + loop-closure retrieval using the prior trajectory — no all-pairs matching. **TensorRT-accelerated ALIKED features** replace SIFT. **Two-stage factor graph** in Ceres: robust-loss warmup → stringent-outlier final pass, decoupled so noisy init doesn't lock in bad local minima. **Multi-camera rig refinement**: joint vehicle pose + camera-to-vehicle transform optimization — the first open-source SfM handling the autonomous-driving rig pattern.

## Why it wins

6× total over COLMAP, 20× mapping-only on KITTI 100-frame runs. ATE 0.040m vs COLMAP 0.406m (90.1% improvement) — the prior-trajectory + two-stage schedule is the load-bearing combination.

## Preconditions & compatibility

Cannot start from scratch (requires prior SLAM trajectory). Rig is rigid — time-varying calibration unsupported. Bet #009 tests generalization outside autonomous driving (drone, phone, AR) where the "refine a coarse trajectory" pattern should still apply.

## Open questions

- Does the two-stage schedule help InstantSfM's general-purpose lane too?
- Handheld/AR IMU noise vs. vehicle IMU: is the prior good enough?
