---
title: Spherical Harmonics
type: concept
tags: [appearance, view-dependent, color, basis-functions]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/sun2025_sparse-voxels-rasterization.md
  - papers/xie2025_gauss-mi.md
  - papers/zhu2025_gs-discretized-sdf.md
  - papers/guo2025_ea-3dgs.md
status: stub
---

## What it is

Spherical harmonics (SH) are a set of orthonormal basis functions defined on the unit sphere. In 3D reconstruction and rendering, they provide a compact, smooth representation of view-dependent color: each point or Gaussian stores a small set of SH coefficients per color channel, and the outgoing radiance for any viewing direction is obtained by evaluating the SH basis at that direction.

## How it works

The SH basis functions are real-valued polynomials of the unit direction vector, organized by degree l and order m. Degree-0 is a constant (diffuse), degree-1 captures directional variation, and higher degrees encode specular-like effects. In 3DGS, each Gaussian typically stores SH coefficients up to degree 3 (16 coefficients per channel). The final color is a dot product of the coefficient vector with the SH basis evaluated at the viewing direction. This representation is differentiable and efficient, making it the standard choice for view-dependent appearance in splatting-based methods.

## Key references

- [Sun 2025](../papers/sun2025_sparse-voxels-rasterization.md) · [pdf](../../papers/radiance-fields/sun_2025_sparse-voxels-rasterization.pdf)
- [Xie 2025](../papers/xie2025_gauss-mi.md) · [pdf](../../papers/radiance-fields/xie_2025_gauss-mi.pdf)
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md) · [pdf](../../papers/radiance-fields/zhu_2025_gs-discretized-sdf.pdf)
