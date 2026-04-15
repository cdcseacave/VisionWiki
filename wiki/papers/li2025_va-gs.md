---
title: "VA-GS: Enhancing the Geometric Representation of Gaussian Splatting via View Alignment"
type: paper
tags: [gaussian-splatting, surface-reconstruction, multi-view-consistency, novel-view-synthesis, geometric-constraints]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/li_2025_va-gs.pdf
url: https://arxiv.org/abs/2510.11473
code: https://github.com/LeoQLi/VA-GS
license_code: none
license_paper: CC-BY-NC-SA-4.0
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/li_2025_va-gs.pdf) · [arXiv](https://arxiv.org/abs/2510.11473) · [code](https://github.com/LeoQLi/VA-GS)

_Code license: `none`_

## TL;DR

VA-GS improves surface reconstruction from [[3d-gaussian-splatting]] by introducing multi-faceted view alignment constraints: edge-aware rendering losses, visibility-aware photometric alignment, normal-based orientation constraints, and deep feature embeddings for cross-view consistency. The method achieves state-of-the-art performance on DTU, Tanks and Temples, Mip-NeRF 360, and Deep Blending for both surface reconstruction and novel view synthesis.

## Problem

3DGS relies primarily on per-image photometric loss, which is insufficient for accurate surface reconstruction due to: (1) illumination-induced artifacts (shadows, specular highlights) distorting the loss signal, (2) ambiguous surface boundaries causing geometry drift or holes, and (3) lack of explicit cross-view geometric consistency enforcement. Prior Gaussian-based surface methods still struggle with these two persistent challenges.

## Method

VA-GS introduces four complementary geometric constraints on top of standard [[3d-gaussian-splatting]]:

1. **Edge-Aware Image Loss**: Incorporates image edge cues into the rendering loss to improve surface boundary delineation.

2. **Visibility-Aware Photometric Alignment ($\mathcal{L}_p$)**: A multi-view photometric consistency loss that explicitly models occlusions via projection error-based weighting. Pixels with large reprojection errors (occluded or misaligned) are downweighted.

3. **Normal-Based Constraints ($\mathcal{L}_{nc}$, $\mathcal{L}_{ns}$)**: Refine spatial orientation of Gaussians to improve local surface estimation and mitigate lighting ambiguities.

4. **Multi-View Feature Alignment ($\mathcal{L}_f$)**: Uses deep image feature embeddings to enforce cross-view consistency, robust to appearance variation.

The final loss is: $\mathcal{L} = \mathcal{L}_I + \lambda_1 \mathcal{L}_{nc} + \lambda_2 \mathcal{L}_{ns} + \lambda_3 \mathcal{L}_p + \lambda_4 \mathcal{L}_f$.

Mesh extraction uses [[tsdf|TSDF]] fusion from rendered depth maps across training views.

## Results

- **DTU**: Achieves the lowest average Chamfer distance among all compared methods, outperforming implicit approaches and all prior Gaussian-based methods.
- **Tanks and Temples**: Best F1-score among all approaches, including both implicit and explicit methods.
- **Mip-NeRF 360**: Highest average PSNR and SSIM among Gaussian-based methods for novel view synthesis.
- **Deep Blending**: Effective handling of complex lighting conditions and ambiguous boundaries.

## Why it matters

VA-GS shows that carefully designed multi-view geometric constraints can substantially close the gap between Gaussian Splatting and implicit neural surface methods for reconstruction quality, while maintaining the rendering efficiency of 3DGS. It demonstrates that the key bottleneck is not the representation itself but the supervision signal.

## Pipeline contribution

- **Visibility-aware photometric alignment $\mathcal{L}_p$ (N1)** — reprojection-error-weighted multi-view photometric loss, downweighting occluded/misaligned pixels. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A · stage: *multi-view photometric loss* · replaces/augments: *per-image photometric loss* · expected gain: removes shadow/specular artifacts; SOTA DTU Chamfer + T&T F1.
- **Edge-aware image loss (N2)** — image-edge cues in the rendering loss. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *boundary supervision* · expected gain: sharp surface boundaries.
- **Normal orientation + smoothing constraints $\mathcal{L}_{nc}/\mathcal{L}_{ns}$ (N3)** — refines Gaussian spatial orientation. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *surface orientation regularization* · expected gain: robust to lighting ambiguity.
- **Multi-view deep-feature alignment $\mathcal{L}_f$ (N4)** — cross-view feature consistency via deep embeddings. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *photometrically-robust cross-view loss* · synthesis-bet: *swap VA-GS's generic deep features for DINOv3 dense features* (from [[simeoni2025_dinov3]]) — should make $\mathcal{L}_f$ more robust on aerial/drone captures with appearance drift.
- **Role**: VA-GS is the **four-constraint Paradigm-A SOTA** as of 2025; together with PGSR's planar constraint + Kim 2025's MVS supervision + CoMe's confidence, forms the stacked-regularization synthesis bet in the thread.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]] and follows the pipeline of methods like [[2d-gaussian-splatting]], GOF, PGSR, RaDe-GS.
- Contrasts with GS-Pull (which only reconstructs foreground) and SuGaR.
- Compared to [[neural-implicit-surfaces]] methods: NeuS, Neuralangelo, VolSDF.
- Uses [[tsdf|TSDF]] for mesh extraction, following standard practice from prior Gaussian surface methods.

## Open questions / limitations

- Feature alignment loss ($\mathcal{L}_f$) shows limited benefit on Tanks and Temples (diverse lighting) compared to DTU (cleaner conditions); stronger feature extractors may help.
- Requires significant GPU resources for training.
- High-fidelity reconstruction raises ethical concerns about unauthorized digital replication.

## Code & license

Repo has **no LICENSE file** — all rights reserved by default under GitHub's terms. Code is viewable but not legally forkable or redistributable without the author's explicit permission.

## References added to the wiki

- [[[li2025_va-gs]]] (this page)
