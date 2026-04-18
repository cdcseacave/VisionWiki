---
title: Multi-resolution voxel-anchor 3DGS SLAM with progressive submaps + loop closure
type: idea
source_paper: wiki/papers/deng2026_vpgs-slam.md
also_in: []

scope: new-paradigm
stages: [radiance-fields.incremental-capture, radiance-fields.global-consistency]
inputs: [rgbd-stream, camera-poses]
outputs: [multi-submap-3dgs, loop-closed-pose-graph]
assumptions: [rgbd-input-available, online-streaming, static-or-quasi-static-scene]
requires_upstream_property: [depth-sensor-or-dense-depth-completion]
requires_downstream_property: [renderer-accepts-submap-union]
learned_params: [per-anchor-mlp-weights, gaussian-attributes]
failure_modes: [monocular-only-unsupported, no-full-global-ba]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs-slam, voxel-anchors, loop-closure, submaps]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Multi-resolution voxel grid anchors carry per-anchor features; Gaussian attributes are decoded from anchor features via small MLPs. Dynamic submap allocation: each submap is O(N)-memory bounded, so as the map grows, new submaps are allocated rather than inflating a global model. Camera tracking is 2D-3D fusion (coarse photometric + fine voxel ICP, adaptive fallback). Loop closure combines 2D rendering loss + 3D voxel ICP + online-distillation submap fusion.

## Why it wins

6× lower map memory + 2–3× lower GPU usage than SplaTAM. Globally-consistent city-scale 3DGS SLAM — SplaTAM and MonoGS lack loop closure; VPGS-SLAM is the first 3DGS SLAM with closed loops at scale.

## Preconditions & compatibility

RGB-D required (or dense-depth completion like DepthLab). No full global BA — loop closure is the only global step. Bundles naturally with VastGaussian's partitioning (Bet #002).

## Pipeline-shape implications

New-paradigm scope: moves from "reconstruct offline then view" to "reconstruct online with loop closure." Threads adopting this change their capture-time topology, not just a single stage.
