---
title: Neural Radiance Fields (NeRF)
type: method
tags: [radiance-fields, neural-rendering, volume-rendering, implicit]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/chebbi2025_multiview-dense-matching.md
  - papers/deng2026_vpgs-slam.md
  - papers/gao2025_anisdf.md
  - papers/guedon2025_milo.md
  - papers/guo2025_ea-3dgs.md
  - papers/jin2026_zipmap.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/li2025_geosvr.md
  - papers/li2025_va-gs.md
  - papers/park2023_camp.md
  - papers/radl2025_sof.md
  - papers/radl2026_confidence-mesh-3dgs.md
  - papers/sun2025_sparse-voxels-rasterization.md
  - papers/tang2025_dronesplat.md
  - papers/yu2025_cusfm.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhong2026_instantsfm.md
  - papers/zhu2025_gs-discretized-sdf.md
status: stub
---

## What it is

Neural Radiance Fields (NeRF), introduced by Mildenhall et al. (ECCV 2020), represent a scene as a continuous volumetric function mapping 3D coordinates and viewing direction to density and color. An MLP encodes this function and is optimized per-scene from posed images via differentiable volume rendering. NeRF established the paradigm of neural implicit 3D reconstruction and remains a foundational reference point for virtually all subsequent neural rendering methods.

## How it works

Rays are cast from camera pixels into the scene. Points are sampled along each ray and queried through the MLP to obtain (density, RGB). Classical volume rendering accumulates these samples into a pixel color. The photometric reconstruction loss trains the MLP end-to-end. Positional encoding of inputs enables the network to capture high-frequency detail. Many follow-up works accelerate training (Instant-NGP), improve quality (Mip-NeRF), or remove the need for known poses.

## Key references

- [Park 2023](../papers/park2023_camp.md) · [pdf](../../papers/radiance-fields/park_2023_camp.pdf)
- [Gao 2025](../papers/gao2025_anisdf.md) · [pdf](../../papers/mesh-reconstruction/gao_2025_anisdf.pdf)
- [Guedon 2025](../papers/guedon2025_milo.md) · [pdf](../../papers/mesh-reconstruction/guedon_2025_milo.pdf)
- [Li 2025 (VA-GS)](../papers/li2025_va-gs.md) · [pdf](../../papers/mesh-reconstruction/li_2025_va-gs.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
