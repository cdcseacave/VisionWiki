---
title: "Zip-NeRF: Anti-Aliased Grid-Based Neural Radiance Fields"
type: paper
tags: [nerf, anti-aliasing, radiance-fields, hash-grid]
created: 2026-04-14
updated: 2026-04-15
sources: [papers/barron2022_mip-nerf-360.md]
local_paper: papers/radiance-fields/barron_2023_zip-nerf.pdf
url: https://arxiv.org/abs/2304.06706
status: stable
---

📄 [Full paper](../../papers/radiance-fields/barron_2023_zip-nerf.pdf) · [arXiv](https://arxiv.org/abs/2304.06706) · [Project](https://jonbarron.info/zipnerf/)

## TL;DR

Barron, Mildenhall, Verbin, Srinivasan & Hedman (ICCV 2023) combine **Mip-NeRF 360's anti-aliased cone-casting** with **Instant-NGP's hash-grid speed**, producing a radiance field that is both fast to train (~1 hour) and visually sharper than either parent — **8× lower error than Mip-NeRF 360** at comparable quality settings.

## Problem

Mip-NeRF 360 rendered well but relied on a slow MLP backbone. Instant-NGP used a hash-grid MLP for speed but aliased badly under scale changes because it point-samples features at scene coordinates, ignoring the footprint of each pixel's cone.

## Method

- **Multisample anti-aliasing**: approximate the cone with a cluster of isotropic-Gaussian samples; query the hash grid at each and average — an anti-aliased reading of the grid.
- **Z-aliasing fix**: normalize the proposal-network's weights across scales to suppress the "jitter along the ray" artifact of grid-based proposal sampling.
- **Retains** Mip-NeRF 360's proposal network, distortion loss, and scene contraction.

## Results

- 8× reduction in error vs. Mip-NeRF 360 on 360° unbounded scenes.
- ~22× faster to train than Mip-NeRF 360; within 2× of Instant-NGP's training time.
- Dramatically reduces "floaters" and temporal jitter in rendered videos.

## Why it matters

Establishes the quality ceiling for *NeRF-family* methods pre-3DGS. Widely used as a baseline in 2024–2026 radiance-field and feed-forward novel-view-synthesis work (see [[radiance-field-evolution]]). Crystallizes "anti-aliasing is a first-class concern for grid-based representations."

## Relation to prior work

- Builds on [Mip-NeRF 360 (Barron 2022)](barron2022_mip-nerf-360.md) (scene contraction, proposal sampling, distortion loss) and Instant-NGP (multi-resolution hash grid).
- Contrasts with [[3d-gaussian-splatting]]: Zip-NeRF wins on quality on reflective/unbounded scenes; 3DGS wins on rendering speed by orders of magnitude.

## Open questions / limitations

- Still 1–2 orders of magnitude slower to render than 3DGS.
- Z-aliasing fix is empirical; no principled analysis of why normalized weights eliminate the artifact.

## References added to the wiki

- [[zip-nerf]] (stub expanded).
