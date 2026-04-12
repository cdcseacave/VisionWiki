---
title: Marching Cubes
type: method
tags: [surface-extraction, mesh, isosurface, classic]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/gao2025_anisdf.md
  - papers/guedon2025_milo.md
  - papers/li2025_geosvr.md
  - papers/radl2025_sof.md
  - papers/radl2026_confidence-mesh-3dgs.md
  - papers/sun2025_sparse-voxels-rasterization.md
status: stub
---

## What it is

Marching Cubes (Lorensen & Cline, SIGGRAPH 1987) is the standard algorithm for extracting a triangular mesh from an implicit function (e.g., a signed distance field or occupancy grid). It processes a 3D voxel grid, classifying each cube vertex as inside or outside the surface, and generates triangles at the boundary using a lookup table of 256 cases.

## How it works

The algorithm iterates over each cell in a regular 3D grid. For each cell, the sign of the implicit function at the eight corners determines a configuration index. A precomputed table maps this index to a set of triangle vertices, which are placed along edges via linear interpolation. The result is a watertight mesh approximating the zero-level-set of the field. Variants like Marching Cubes 33 and dual contouring improve topological correctness and sharp-feature preservation.

## Key references

- [Gao 2025](../papers/gao2025_anisdf.md) · [pdf](../../papers/mesh-reconstruction/gao_2025_anisdf.pdf)
- [Guedon 2025](../papers/guedon2025_milo.md) · [pdf](../../papers/mesh-reconstruction/guedon_2025_milo.pdf)
- [Radl 2025](../papers/radl2025_sof.md) · [pdf](../../papers/mesh-reconstruction/radl_2025_sof.pdf)
- [Radl 2026](../papers/radl2026_confidence-mesh-3dgs.md) · [pdf](../../papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf)
- [Li 2025 (GeoSVR)](../papers/li2025_geosvr.md) · [pdf](../../papers/mesh-reconstruction/li_2025_geosvr.pdf)
