---
title: Radiance-fields importance sampling
type: stage
slug: radiance-fields.importance-sampling
consumes: [per-ray-coarse-density-or-weight, large-renderer-mlp]
produces: [concentrated-sample-positions, normalized-proposal-weights]
invariants: [sample-concentration-respects-coarse-signal]
provides_properties: [cheap-proposal-high-quality-main-pass]
requires_upstream_properties: [main-renderer-differentiable]
data_regime: [ray-march, nerf-family]
tags: [nerf, proposal-network, distillation, zip-nerf]
created: 2026-04-18
updated: 2026-04-18
---

Selects where along each ray to sample densely. Online-distilled proposal MLP (Mip-NeRF 360) + Z-aliasing normalization (Zip-NeRF). NeRF-family only; 3DGS has no analog (rasterization concentrates work by construction). Example fillers: [[proposal-network-online-distillation_barron2022]], [[z-aliasing-normalization_barron2023]].
