---
title: 2D Gaussian Splatting
type: method
tags: [radiance-fields, surface-reconstruction, gaussian-splatting]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/guedon2025_milo.md
  - papers/kim2025_multiview-geometric-gs.md
  - papers/radl2026_confidence-mesh-3dgs.md
status: stub
---

## What it is

2D Gaussian Splatting (2DGS), introduced by Huang et al. (2024), replaces the 3D ellipsoidal Gaussians of 3DGS with flat 2D Gaussian disks (surfels). By constraining each primitive to lie on a tangent plane, 2DGS produces geometry that is more surface-aligned, yielding cleaner mesh extraction compared to standard 3DGS.

## How it works

Each 2D Gaussian is parameterized by a center, a normal direction, and two tangent-plane scale/rotation parameters. During rasterization the disk is splatted onto the image plane similarly to 3DGS, but the intersection of the ray with the disk plane provides a well-defined depth. This geometric consistency makes 2DGS particularly suitable for downstream mesh extraction via TSDF fusion or marching cubes.

## Key references

- [Guedon 2025](../papers/guedon2025_milo.md) · [pdf](../../papers/mesh-reconstruction/guedon_2025_milo.pdf)
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md) · [pdf](../../papers/radiance-fields/kim_2025_multiview-geometric-gs.pdf)
- [Radl 2026](../papers/radl2026_confidence-mesh-3dgs.md) · [pdf](../../papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf)
