---
title: Neural Implicit Surfaces
type: concept
tags: [sdf, implicit-representation, reconstruction, volume-rendering]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/gao2025_anisdf.md, papers/li2025_geosvr.md, papers/li2025_va-gs.md, papers/elflein2026_vgg-t3.md]
status: stub
---

## What it is
A family of 3D surface representations that encode geometry as the zero level-set of a neural [[signed-distance-field|SDF]] (or occupancy field), rendered via volume integration. Trades the speed of explicit representations like [[3d-gaussian-splatting]] for smoother, watertight surfaces.

## Lineage
- IDR → [[neus|NeuS]] → VolSDF → NeuS2 → Neuralangelo → AniSDF
- Hybrid with Gaussians: GSDF, GS-Pull, SuGaR

## Strengths vs. limitations
- **+** High-quality continuous surfaces, good for mesh extraction.
- **−** Slower to train/render than explicit methods; mesh quality sensitive to regularization.
