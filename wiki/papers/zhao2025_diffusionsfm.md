---
title: "DiffusionSfM: Predicting Structure and Motion via Ray Origin and Endpoint Diffusion"
type: paper
tags: [sfm, diffusion, transformer, ray-representation, sparse-view, pose-estimation, 3d-reconstruction]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/zhao_2025_diffusionsfm.pdf
url: https://arxiv.org/abs/2505.05473
status: draft
---

📄 [Full paper](../../papers/sfm-slam/zhao_2025_diffusionsfm.pdf) · [arXiv](https://arxiv.org/abs/2505.05473)

## TL;DR

DiffusionSfM is an end-to-end multi-view model that directly infers dense 3D geometry and camera poses from sparse (2-8) input images using a denoising diffusion process. It parameterizes scenes as pixel-wise ray origins (camera centers) and endpoints (3D surface points) in a global frame, unifying structure and motion prediction into a single transformer-based diffusion model that eliminates the need for separate pairwise reasoning and global optimization stages.

## Problem

Traditional [[structure-from-motion]] pipelines (both classical and learning-based like [[dust3r|DUSt3R]]) follow a two-stage approach: pairwise reasoning followed by global optimization. Even [[dust3r|DUSt3R]] and [[mast3r|MASt3R]], while predicting 3D pointmaps directly, still require expensive global alignment for more than two views. Methods like RayDiffusion predict only camera poses without scene structure. No existing approach unifies multi-view reasoning for both structure and motion in a single end-to-end framework.

## Method

DiffusionSfM uses a Diffusion Transformer (DiT) architecture:

1. **Ray Origin and Endpoint Representation**: Each pixel is associated with a ray origin $O_{ij}$ (camera center) and endpoint $E_{ij}$ (3D surface point) in world coordinates. Origins are predicted densely (providing implicit regularization) and can be converted back to traditional camera extrinsics/intrinsics.

2. **Diffusion Framework**: Forward process adds Gaussian noise to ray origins and endpoints; reverse process iteratively denoises using a transformer conditioned on DINOv2 image features. A DPT decoder produces full-resolution outputs.

3. **Key Training Mechanisms**:
   - **Homogeneous coordinates**: $(x,y,z) \to \frac{1}{w}(x,y,z,1)$ with unit normalization bounds unbounded scene geometry for stable diffusion training.
   - **GT mask conditioning**: Binary masks inform the model about missing depth data during training; set to all-ones at inference for dense prediction.
   - **Sparse-to-dense training**: First trains a sparse (patch-resolution) model, then initializes the dense model from learned weights for faster convergence.

Multi-view reasoning happens through self-attention across all image patches with sinusoidal positional encoding for image and patch indices.

## Results

- **CO3D (seen categories)**: Camera rotation accuracy @15° of 95.5 (8 views) vs. DUSt3R 94.3, RayDiffusion 93.3. Camera center accuracy @0.1 of 81.6 vs. DUSt3R 71.6, RayDiffusion 73.7.
- **CO3D (unseen categories)**: Rotation accuracy 90.7 (8 views) vs. DUSt3R 76.8, demonstrating generalization.
- **Habitat (scene-level)**: Camera center accuracy @0.3 of 91.3 vs. DUSt3R 81.1, COLMAP 12.7. Chamfer distance 0.198 vs. DUSt3R 0.291.
- **RealEstate10K**: Camera center accuracy @0.2 of 70.3 vs. DUSt3R 35.9.
- Naturally models uncertainty through the diffusion process; multiple samples capture pose ambiguity.

## Why it matters

DiffusionSfM represents a paradigm shift from the traditional two-stage SfM pipeline (pairwise + global optimization) to a single-pass multi-view inference model. The diffusion formulation provides built-in uncertainty modeling, which is valuable when inputs are ambiguous. The ray origin/endpoint parameterization elegantly unifies camera and geometry prediction in a distributed representation compatible with modern vision backbones.

## Relation to prior work

- Extends RayDiffusion's ray-based camera representation to include endpoints (geometry), going from patch-wise to pixel-wise resolution.
- Competes with [[dust3r|DUSt3R]] and [[mast3r|MASt3R]] but eliminates global alignment; processes N views in one pass rather than O(N^2) pairwise predictions.
- Uses [[DINOv2]] backbone features, following the trend of leveraging foundation model features for 3D tasks.
- Related to PoseDiffusion (diffusion for pose estimation only) and ACEZero/FlowMap (unified SfM approaches with incremental processing).
- DPT decoder follows standard dense prediction transformer design.

## Open questions / limitations

- Limited to 2-8 input views due to quadratic self-attention within the transformer; does not scale to large image collections.
- Trained primarily on object-centric (CO3D) and indoor/outdoor datasets; performance on diverse in-the-wild scenarios not extensively evaluated.
- The diffusion sampling process requires multiple denoising steps, adding latency compared to single-pass regression models.
- No explicit handling of dynamic scenes or moving objects.

## References added to the wiki

- [[structure-from-motion]]
- [[dust3r|DUSt3R]]
- [[mast3r|MASt3R]]
- [[diffusion-models]]
- [[DINOv2]]
- [[DPT]]
- [[bundle-adjustment]]
- [[RayDiffusion]]
- [[colmap|COLMAP]]
- [[homogeneous-coordinates]]
