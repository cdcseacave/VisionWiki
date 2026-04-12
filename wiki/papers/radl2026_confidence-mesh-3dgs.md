---
title: "Confidence-Based Mesh Extraction from 3D Gaussians"
type: paper
tags: [gaussian-splatting, mesh-extraction, confidence-estimation, surface-reconstruction, appearance-modeling]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf
url: https://arxiv.org/abs/2603.24725
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/radl_2026_confidence-mesh-3dgs.pdf) · [arXiv](https://arxiv.org/abs/2603.24725)

## TL;DR

This paper introduces a self-supervised confidence framework for [[3d-gaussian-splatting]] mesh extraction, where learnable per-Gaussian confidence values dynamically balance photometric and geometric supervision. Combined with per-primitive color/normal variance losses and an improved SSIM-decoupled appearance model, the method achieves state-of-the-art unbounded mesh extraction quality in ~20 minutes, outperforming MILo (60 min) and other baselines.

## Problem

Mesh extraction from 3DGS remains difficult in scenes with abundant view-dependent effects (specular highlights, reflections). The fundamental challenge is the coupling of geometry and appearance: high-frequency view-dependent effects are often represented as highly opaque Gaussians behind semi-transparent surfaces, degrading geometric accuracy. Prior works resolve these ambiguities using multi-view techniques, iterative mesh extraction, or large pre-trained models, all of which sacrifice the inherent efficiency of 3DGS.

## Method

The method introduces several complementary components:

1. **Self-Supervised Confidence Loss ($\mathcal{L}_{conf}$)**: Each Gaussian is equipped with a learnable confidence value that dynamically balances the influence of photometric vs. geometric losses during optimization. High-confidence regions receive stronger geometric supervision; low-confidence regions (view-dependent effects) are supervised more by photometric loss.

2. **Per-Primitive Variance Losses**: Penalize per-primitive color variance ($\mathcal{L}_{color\text{-}var}$) and normal variance ($\mathcal{L}_{normal\text{-}var}$) across views, constraining every primitive to be aligned with object surfaces and removing spurious geometry.

3. **SSIM-Decoupled Appearance Model**: Decomposes the D-SSIM loss into luminance, contrast, and structure terms ($l \cdot c \cdot s$). The appearance embedding compensates only for luminance, while contrast and structure terms use the original rendered image. This prevents the appearance model from masking geometric errors.

Mesh extraction follows the SOF pipeline using [[marching-cubes]] on the opacity field.

## Results

- **Tanks and Temples**: Leading average F1-score of 0.521, outperforming all baselines by a significant margin. Completes in ~20 minutes.
- **ScanNet++**: Best F1-score across all scenes, demonstrating real-world applicability under exposure variation, inconsistent lighting, and blur.
- **Mip-NeRF 360**: Adding confidence does not negatively impact NVS metrics (PSNR drops by merely 0.02 dB) while reducing primitive count by 26%.
- Indoor scenes show higher average confidence (3.22) than outdoor scenes (1.66), with image quality improvement directly correlated to confidence values.

## Why it matters

This work shows that the key to better mesh extraction from Gaussians is not more complex architectures but smarter loss balancing. The confidence framework is simple, efficient, and self-supervised -- it does not require pre-trained models or multi-view consistency checks at test time. The SSIM decoupling insight is particularly elegant: standard appearance embeddings inadvertently hide geometric errors by compensating for structure/contrast differences.

## Relation to prior work

- Directly builds on **SOF** ([[radl2025_sof]]) for the base pipeline and mesh extraction.
- Compares against MILo ([[guedon2025_milo]]), PGSR, QGS, GOF, [[2d-gaussian-splatting]].
- Contrasts with UA-GS and VCR-GauS which use confidence to balance pseudonormal predictions rather than photometric/geometric losses.
- Appearance model improves upon VastGaussian's widely adopted appearance embedding.
- Uses [[3d-gaussian-splatting]] with [[marching-cubes]] for surface extraction.

## Open questions / limitations

- Confidence hyperparameter beta is the primary tunable parameter; sensitivity analysis shows robust but not completely parameter-free behavior.
- The method focuses on unbounded scenes; performance on small object-centric datasets (DTU) not emphasized.
- Does not address the fundamental SH limitation for high-frequency view-dependent effects.

## References added to the wiki

- [[radl2026_confidence-mesh-3dgs]] (this page)
