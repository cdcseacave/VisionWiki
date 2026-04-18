---
title: Plucker ray-bundle camera representation for transformer pose regression
type: idea
source_paper: wiki/papers/zhang2024_cameras-as-rays.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.camera-parameterization]
inputs: [image-patches, dinov2-features]
outputs: [per-patch-6d-plucker-rays, camera-extrinsics]
assumptions: [dinov2-or-frozen-ssl-backbone, transformer-regression-head]
requires_upstream_property: [ssl-backbone-with-patch-tokens]
requires_downstream_property: [consumer-can-invert-ray-bundle-to-camera]
learned_params: [transformer-weights]
failure_modes: [over-parameterized-head-may-not-regularize-without-implicit-structure]

requires: []
unlocks: [diffusion-over-ray-bundles_zhang2024]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, plucker-rays, pose-regression, transformer]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Each image patch gets a 6D Plucker ray (direction + moment); the collection of `N · p²` rays is the camera representation. Distributed over-parameterized alternative to global (R, t, K). Transformer regresses per-patch rays directly, L2 loss on ray parameters.

## Why it wins

+13% rotation @15°, +23% center @0.1 on CO3D over PoseDiffusion (global-head baseline). Even regression-only beats prior diffusion-based SOTA. Conceptual ancestor of DiffusionSfM's ray-origin+endpoint parameterization and of the per-pixel geometric predictions in DUSt3R/MASt3R/VGGT.

## Open questions

- Why does the over-parameterized distributed representation outperform compact (R,t,K)? Paper hypothesizes implicit spatial smoothing via the transformer's attention pattern — not ablated.
