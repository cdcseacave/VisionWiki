---
title: Signed Distance Field (SDF)
type: concept
tags: [implicit-representation, surface, geometry, neural-implicit]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/gao2025_anisdf.md
  - papers/guedon2025_milo.md
  - papers/li2025_geosvr.md
  - papers/li2025_va-gs.md
  - papers/radl2025_sof.md
  - papers/sun2025_sparse-voxels-rasterization.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhu2025_gs-discretized-sdf.md
status: stub
---

## What it is

A Signed Distance Field (SDF) is an implicit surface representation that assigns to every point in 3D space the signed distance to the nearest surface: negative inside, positive outside, zero on the surface. SDFs are widely used in neural 3D reconstruction because the zero-level-set provides a clean, continuous surface definition amenable to mesh extraction via marching cubes.

## How it works

An SDF can be stored on a voxel grid, encoded in an MLP (as in NeuS, VolSDF), or represented by other data structures (octrees, hash grids). For rendering, sphere tracing or volume rendering with SDF-derived density converts the field into images. Training typically combines a photometric loss with an Eikonal regularizer (encouraging unit-gradient magnitude). The resulting zero-level-set is extracted as a mesh using marching cubes.

## Key references

- [Gao 2025](../papers/gao2025_anisdf.md) · [pdf](../../papers/mesh-reconstruction/gao_2025_anisdf.pdf)
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md) · [pdf](../../papers/radiance-fields/zhu_2025_gs-discretized-sdf.pdf)
- [Li 2025 (GeoSVR)](../papers/li2025_geosvr.md) · [pdf](../../papers/mesh-reconstruction/li_2025_geosvr.pdf)
- [Radl 2025](../papers/radl2025_sof.md) · [pdf](../../papers/mesh-reconstruction/radl_2025_sof.pdf)
