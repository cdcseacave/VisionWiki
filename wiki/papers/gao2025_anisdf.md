---
title: "AniSDF: Fused-Granularity Neural Surfaces with Anisotropic Encoding for High-Fidelity 3D Reconstruction"
type: paper
tags: [neural-implicit-surfaces, SDF, anisotropic-encoding, reflective-surfaces, novel-view-synthesis]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/gao_2025_anisdf.pdf
url: https://arxiv.org/abs/2410.01202
code: https://github.com/G-1nOnly/AniSDF
license_code: MIT
license_paper: arxiv-nonexclusive
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/gao_2025_anisdf.pdf) · [arXiv](https://arxiv.org/abs/2410.01202) · [code](https://github.com/G-1nOnly/AniSDF)

_Code license: `MIT`_

## TL;DR

AniSDF is a unified [[signed-distance-field|SDF]]-based method that combines fused-granularity neural surfaces (coarse + fine hash grid branches) with anisotropic spherical Gaussian (ASG) encoding to achieve simultaneous high-quality geometry reconstruction and photorealistic novel-view synthesis, particularly excelling on reflective and complex objects.

## Problem

Neural radiance fields achieve high-fidelity rendering but sacrifice geometry accuracy, limiting downstream applications like relighting and deformation. Prior neural surface methods (NeuS, Neuralangelo) struggle with: (1) losing fine geometric details due to single-granularity hash grids, where coarse-to-fine training filters out thin structures early on, and (2) incorrectly baking specular reflections into geometry, causing geometric artifacts on reflective objects.

## Method

AniSDF introduces two key components:

1. **Fused-Granularity Neural Surfaces**: Instead of a single coarse-to-fine hash grid, uses two parallel branches -- a coarse-granularity branch (levels 4--10) and a fine-granularity branch (levels 10--16). The coarse branch captures overall structure while the fine branch preserves thin structures and high-frequency details that would otherwise be lost during early training.

2. **Anisotropic Spherical Gaussian (ASG) Encoding**: Models the blended radiance field using physics-based ASG encoding to explicitly decompose appearance into diffuse and specular components. This disambiguates geometry from reflective appearance, following the rendering equation more faithfully than standard view-dependent color prediction.

The surface is extracted via [[marching-cubes]] from the learned [[signed-distance-field|SDF]].

## Results

- **NeRF Synthetic**: Achieves the highest PSNR and lowest Chamfer distance among all volumetric and surface reconstruction methods.
- **DTU**: Competitive Chamfer distance with state-of-the-art SDF-based methods.
- **Shiny Blender**: Best surface reconstruction for reflective objects; successfully handles luminous objects where all other methods fail.
- **Shelly Dataset**: Additional validation on complex specular scenes.
- Training: ~2--3 hours on a single Tesla V100 with 24GB VRAM.

## Why it matters

AniSDF demonstrates that jointly solving geometry and appearance -- rather than treating them separately -- leads to better results in both tasks. The fused-granularity design is a practical architectural insight: parallel coarse and fine branches outperform sequential coarse-to-fine training by preserving fine structures from the start. The physics-based appearance model further shows the value of inductive biases from rendering theory.

## Pipeline contribution

- **Fused-granularity parallel coarse+fine hash-grid branches (N1)** — coarse levels 4–10 + fine levels 10–16, trained jointly rather than coarse-to-fine. candidate thread: [[gaussian-to-mesh-pipelines]] implicit-SDF lane · stage: *feature-grid architecture* · replaces/augments: *single-grid coarse-to-fine (Neuralangelo, NeuS)* · expected gain: preserves thin structures lost in sequential coarse-to-fine training.
- **Anisotropic spherical Gaussian (ASG) radiance decomposition (N2)** — explicit diffuse + specular branch following rendering equation. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *appearance model separating geometry from specularity* · replaces/augments: *view-dependent color MLP that bakes specular into geometry* · expected gain: best Shiny Blender / luminous-object reconstruction; physics-based inductive bias wins on reflective scenes.
- **Role**: AniSDF is the *implicit-SDF alternative lane* for the thread. Most synthesis bets phrased around 3DGS have AniSDF/NeuS-style counterparts — the thread's contradictions ("external MVS vs self-supervised") apply here too.

## Relation to prior work

- Extends [[neural-implicit-surfaces]] methods: NeuS, VolSDF, Neuralangelo, NeuS2.
- Uses multi-resolution hash encoding from Instant-NGP but with a novel dual-branch architecture.
- ASG encoding draws from physics-based rendering and prior work on spherical Gaussians for appearance modeling (Ref-NeuS, NeRO).
- Contrasts with [[3d-gaussian-splatting]] methods that handle appearance but lack explicit [[signed-distance-field|SDF]] geometry.
- Surface extraction via [[marching-cubes]] on the learned [[signed-distance-field|SDF]].

## Open questions / limitations

1. **No real-time rendering**: Could potentially be addressed by SDF baking (as in BakedSDF) adapted for ASG encoding.
2. **Complex indirect illumination**: Fails in cases with complex indirect lighting due to lack of a materials estimation network.
3. Memory: Fine-granularity branch with resolution 4096 requires 32GB; default 2048 uses 24GB.

## References added to the wiki

- [[[gao2025_anisdf]]] (this page)
