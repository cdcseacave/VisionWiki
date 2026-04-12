---
title: Differentiable Rendering
type: concept
tags: [rendering, optimization, gradient, inverse-graphics]
created: 2026-04-12
updated: 2026-04-12
sources:
  - papers/guedon2025_milo.md
  - papers/li2025_megasam.md
  - papers/park2023_camp.md
  - papers/xie2025_gauss-mi.md
  - papers/zhang2025_feed-forward-3d-survey.md
status: stub
---

## What it is

Differentiable rendering is the technique of computing gradients of a rendered image with respect to scene parameters (geometry, appearance, camera). It is the key enabler for optimizing 3D representations from 2D image supervision, underpinning NeRF, 3D Gaussian Splatting, and most neural reconstruction methods.

## How it works

A differentiable renderer implements the forward rendering equation (volume rendering, rasterization, or ray marching) in a way that supports backpropagation. For volume rendering (NeRF), gradients flow through the quadrature approximation of the rendering integral. For splatting (3DGS), gradients flow through the alpha-compositing of projected Gaussians. For mesh-based methods, soft rasterizers or differentiable ray-triangle intersections enable gradient computation. The resulting gradients allow scene parameters to be optimized via standard gradient descent to match observed images.

## Key references

- [Park 2023](../papers/park2023_camp.md) · [pdf](../../papers/radiance-fields/park_2023_camp.pdf)
- [Guedon 2025](../papers/guedon2025_milo.md) · [pdf](../../papers/mesh-reconstruction/guedon_2025_milo.pdf)
- [Xie 2025](../papers/xie2025_gauss-mi.md) · [pdf](../../papers/radiance-fields/xie_2025_gauss-mi.pdf)
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf)
