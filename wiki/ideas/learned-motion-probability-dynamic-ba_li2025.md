---
title: Learned motion-probability maps + uncertainty-aware BA (MegaSaM)
type: idea
source_paper: wiki/papers/li2025_megasam.md
also_in: []

scope: topology-rewrite
stages: [sfm.dynamic-content-masking, sfm.bundle-adjustment]
collapses: []
splits_into: []
rewrites: {replaces: [sfm.bundle-adjustment], introduces: [sfm.motion-segmentation, sfm.uncertainty-aware-ba, sfm.consistent-video-depth-refinement]}

inputs: [casual-video-stream, depth-anything-or-unidepth-init]
outputs: [posed-dynamic-video, consistent-depth-at-full-res]
assumptions: [dynamic-content-not-dominating-image, monocular-prior-available]
requires_upstream_property: [depth-anything-init-available, optical-flow-available]
requires_downstream_property: [consumer-handles-casual-video-output]
learned_params: [motion-probability-network-weights]
failure_modes: [fails-when-dynamic-dominates-entire-image, cannot-handle-purely-rotational-without-parallax]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, dynamic-scene, monocular-prior, uncertainty-ba, droid-slam-extension]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Two-stage training: **ego-motion pretraining** → **dynamic fine-tuning** with frozen flow. Produces per-pixel motion-probability maps that mask dynamic content out of BA. **Monocular depth initialization** from DepthAnything + UniDepth replaces DROID-SLAM's constant disparity init. **Uncertainty-aware BA**: Hessian-diagonal-driven adaptive regularization — disables unobservable parameters (focal length on near-rotational motion) cleanly. **Consistent depth module**: optical-flow reprojection + temporal consistency + normal loss at full res, no network fine-tune (CasualSAM quality at 100× speed).

## Why it wins

Casual dynamic videos become tractable: ATE 0.018 on Sintel vs 0.036 next-best. Four mechanisms address four failure modes (dynamic content, poor init, unobservable params, per-video depth refinement); removing any one hurts.

## Pipeline-shape implications

Topology-rewrite: adds motion-segmentation + uncertainty-aware-ba + depth-refinement stages. Replaces single BA node with three. Threads adopting this must represent the expanded DAG.

## Open questions

- Fails when moving objects dominate or no static scene to track. Hard failure, not graceful.
