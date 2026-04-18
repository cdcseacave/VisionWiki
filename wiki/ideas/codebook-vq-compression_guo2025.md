---
title: Codebook VQ compression + tetrahedral-mesh init for 3DGS
type: idea
source_paper: wiki/papers/guo2025_ea-3dgs.md
also_in: []

scope: drop-in
stages: [radiance-fields.compression]
inputs: [trained-3dgs, non-position-attributes]
outputs: [quantized-3dgs, learned-codebook]
assumptions: [post-training-compression-OK, commercial-license-blocked-3dgs-base]
requires_upstream_property: [trained-3dgs-available]
requires_downstream_property: [renderer-accepts-codebook-indexed-attributes]
learned_params: [codebook-entries, per-gaussian-indices]
failure_modes: [quantization-requires-separate-fine-tuning-phase, no-QAT-support]

requires: []
unlocks: []
co_requires: [adaptive-tetrahedral-init_guo2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, compression, vq, city-scale]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Post-training: learn a codebook over non-position attributes (SH coefficients, scales, rotations, opacities). Each Gaussian stores indices into the codebook instead of raw attributes. Adaptive tetrahedral-mesh initialization (Delaunay over COLMAP sparse points, Gaussians spawned on triangular faces) fills textureless regions before compression so the codebook is trained on a representative distribution.

## Why it wins

5.2× compression (1.8 GB → 345 MB) at 0.37 dB PSNR cost. Orthogonal to spatial partitioning (VastGaussian) and incremental SLAM (VPGS-SLAM) — layering all three is Bet #002.

## Preconditions & compatibility

Commercial license blocked (inherits Gaussian-Splatting-License). Tetra-init adds preprocessing time; scalability to very large point clouds may be an issue. Codebook requires a separate fine-tuning phase — end-to-end quantization-aware training is the natural extension.

## Open questions

- Dynamic-object handling not addressed.
- Can the codebook be shared across submaps in a SLAM setting?
