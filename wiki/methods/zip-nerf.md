---
title: Zip-NeRF
type: method
tags: [nerf, anti-aliasing, radiance-fields]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/barron2023_zip-nerf.md]
status: draft
---

## What it is

Zip-NeRF (Barron et al. 2023) combines the **grid-based speed** of Instant-NGP with the **anti-aliasing** of Mip-NeRF 360, producing a fast, high-quality unbounded-scene radiance field.

## Key ideas
- Multi-sample cone casting over hash-grid features (anti-aliased grid sampling).
- Normalized-weight interpolation across scales to suppress z-aliasing.
- Proposal-network distortion loss from Mip-NeRF 360 retained.

## Relation to prior work
- Builds on Mip-NeRF 360 (scene contraction, proposal sampling) and Instant-NGP (hash grid).
- Frequent baseline for 2024–2026 radiance-field and feed-forward NVS papers (see [[radiance-field-evolution]]).

## Why it matters
Became the de facto "quality ceiling" NeRF baseline until 3DGS shifted the comparison axis to rasterization-based methods.

## Key references
- [Barron et al. 2023](../papers/barron2023_zip-nerf.md) · [pdf](../../papers/radiance-fields/barron_2023_zip-nerf.pdf) — canonical Zip-NeRF paper, ICCV 2023.
