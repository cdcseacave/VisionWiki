---
title: Multisample anti-aliased hash-grid query (Zip-NeRF)
type: idea
source_paper: wiki/papers/barron2023_zip-nerf.md
also_in: []

scope: stage-swap
stages: [radiance-fields.grid-feature-sampling]
inputs: [per-ray-cone, hash-grid]
outputs: [anti-aliased-feature-sample]
assumptions: [hash-grid-based-representation, cone-cast-rendering]
requires_upstream_property: [hash-grid-representation-available]
requires_downstream_property: [renderer-uses-multisampled-features]
learned_params: [hash-grid-entries]
failure_modes: [inherits-hash-grid-collision-artifacts]

requires: []
unlocks: []
co_requires: [z-aliasing-normalization_barron2023]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, hash-grid, anti-aliasing, zip-nerf]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Replace Instant-NGP's point sample with a cluster of isotropic Gaussian samples approximating the per-ray cone; query the hash grid at each, average features. The multisample pattern is parameterized to match the cone's footprint in texture space.

## Why it wins

8× lower error vs. Mip-NeRF 360, no aliasing on scale changes. The last major quality win for NeRF-family methods before 3DGS — Zip-NeRF's 22× speedup over Mip-NeRF 360 is what made the NeRF→3DGS handoff a fair fight (3DGS won on edit-ability + render speed, not raw quality).

## Preconditions & compatibility

Bundles with Z-aliasing normalization (`co_requires:`) — multisampling alone leaves along-ray jitter that the Z-aliasing fix cleans up.
