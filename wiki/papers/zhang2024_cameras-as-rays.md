---
title: "Cameras as Rays: Pose Estimation via Ray Diffusion"
type: paper
tags: [pose-estimation, camera-rays, diffusion-model, sparse-view, transformer, plucker-coordinates]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/pose-estimation/zhang_2024_cameras-as-rays.pdf
url: https://arxiv.org/abs/2402.14817
status: draft
---

📄 [Full paper](../../papers/pose-estimation/zhang_2024_cameras-as-rays.pdf) · [arXiv](https://arxiv.org/abs/2402.14817)

## TL;DR

This paper proposes representing cameras as bundles of rays (using Plucker coordinates) instead of the traditional compact (R, t, K) parameterization for learning-based pose estimation. Each image patch is associated with a 6D ray, creating a distributed, over-parameterized representation naturally suited for transformer-based architectures. A regression-based approach already surpasses prior state-of-the-art, and extending it to a denoising diffusion model further improves performance by capturing inherent ambiguities in sparse-view pose inference.

## Problem

Estimating camera poses from sparsely sampled views (<10 images) is challenging because SfM methods fail with insufficient overlap. Existing learning-based methods predict global camera parametrizations (rotation + translation), but this parsimonious representation may be suboptimal for neural networks, which often benefit from over-parameterized distributed representations. Global predictions also cannot easily leverage spatial patch-level correspondences.

## Method

1. **Ray representation**: A camera is represented as a bundle of $m = p^2$ rays over a uniform grid:
   - Each ray $r = \langle d, m \rangle \in \mathbb{R}^6$ uses Plucker coordinates where $d$ is direction and $m = p \times d$ is the moment vector
   - Converting camera to rays: $d = R^\top K^{-1} u$, $m = (-R^\top t) \times d$
   - Converting rays back to camera: solve for center $c$ (least-squares intersection), then recover $K, R$ via DLT and RQ decomposition

2. **Regression approach**: A spatial feature extractor (DINOv2) produces per-patch tokens, concatenated with normalized pixel coordinates. A transformer jointly processes all $N \cdot p^2$ tokens and predicts rays per patch. Trained with L2 reconstruction loss on rays.

3. **Diffusion approach (RayDiffusion)**: Extends regression to a denoising diffusion model over the ray representation:
   - Forward process adds Gaussian noise to ground-truth rays
   - Denoiser network takes spatial features concatenated with noisy rays (visualized as 3-channel direction and moment maps)
   - Uses DDPM with 1000 timesteps during training, 50-step DDIM for inference
   - Can sample multiple modes to handle ambiguities from symmetry and partial observations

4. **Architecture**: Transformer with 12 blocks, hidden dim 768, processing concatenated image features and ray representations.

## Results

- **CO3D dataset (seen categories)**: RayDiffusion achieves camera rotation accuracy @15 of 75.5% and camera center accuracy @0.1 of 56.4%, substantially outperforming PoseDiffusion (62.6% / 33.3%), RelPose++ (70.3% / 44.0%), and SparsePose (54.0% / 14.4%).
- **CO3D (unseen categories)**: RayDiffusion generalizes well, achieving 67.7% rotation accuracy @15 and 50.1% center accuracy @0.1, again outperforming all baselines.
- **Regression alone beats prior art**: Even without diffusion, the regression-based ray prediction (73.3% rotation @15) surpasses all previous methods.
- **In-the-wild generalization**: Qualitative results on unseen datasets and self-captures demonstrate practical applicability.
- **Inference**: 50-step DDIM sampling; multiple samples can be drawn to capture distribution modes.

## Why it matters

This work fundamentally rethinks camera pose representation for learning-based estimation. The ray bundle representation is naturally suited for transformers (each ray maps to a patch token), enables tight coupling with spatial image features, and can encode non-perspective cameras (e.g., catadioptric, orthographic). The idea of distributed camera representations has influenced subsequent works like [[dust3r|DUSt3R]] and [[Pow3R]] which also use per-pixel geometric predictions rather than global camera parameters.

## Pipeline contribution

- **Plucker ray-bundle camera representation (N1)** — each image patch gets a 6D ray; distributed over-parameterized alternative to (R,t,K). candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *camera parameterization for transformer regression* · replaces/augments: *global (R,t,K) heads* · expected gain: +13% rotation @15°, +23% center @0.1 on CO3D over PoseDiffusion.
- **Regression over ray tokens (N2)** — transformer processes $N \cdot p^2$ tokens; L2 on rays. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *pose head* · expected gain: even regression-only beats prior diffusion-based SOTA.
- **RayDiffusion (denoising over ray bundles) (N3)** — diffusion posterior captures symmetry/partial-observation ambiguities. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *uncertainty-aware pose output* · expected gain: multi-modal pose distributions under ambiguous inputs.
- **Role**: conceptual ancestor of [zhao2025_diffusionsfm]'s ray-origin+endpoint parameterization; the ray-bundle idea propagated into DUSt3R/MASt3R/VGGT's per-pixel geometric predictions. Directly grounds [[foundation-features-for-geometry]]'s "distributed, frozen-DINOv2-fed task head" template.
- **Synthesis-bet candidate**: *ray-bundle regression head on top of DINOv3* (replacing DINOv2) + *RoMa v2 dense matching initialization as a diffusion-conditioning signal*. Combines this paper's N1/N3 with [simeoni2025_dinov3] + [edstedt2025_roma-v2]. No paper does this.

## Relation to prior work

- Contrasts with global pose prediction in [[RelPose]], [[RelPose++]], [[SparsePose]], and [[PoseDiffusion]]
- Uses [[dinov2|DINOv2]] (ViT-B/14, frozen + registers) as the spatial feature backbone
- Concurrent with [[dust3r|DUSt3R]] which predicts pixel-aligned pointclouds (rather than rays) and uses PnP
- Relates to generalized camera models of Grossberg & Nayar (2001)
- Diffusion component builds on [[DDPM]] / [[DDIM]] frameworks
- Classical SfM methods ([[colmap|COLMAP]], [[bundle-adjustment]]) provide sub-pixel accuracy but require dense views
- [[Neural Ray Surfaces]] (Vasiljevic et al., 2020) also uses ray-based cameras but for video, not sparse views

## Open questions / limitations

- Evaluated primarily on object-centric scenes (CO3D); performance on large-scale outdoor scenes is less explored
- Ray-to-camera conversion assumes perspective cameras; while the ray representation is more general, the recovery step currently targets pinhole models
- Diffusion sampling adds computational cost compared to single-pass regression
- The method does not produce 3D reconstruction -- it focuses purely on camera pose
- Performance degrades for very wide baselines or views with minimal overlap

## References added to the wiki

- [[RayDiffusion]]
- [[cameras-as-rays]]
- [[plucker-coordinates]]
- [[dinov2|DINOv2]]
- [[DDPM]]
- [[sparse-view-pose-estimation]]
