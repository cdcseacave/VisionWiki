---
title: "Advances in Feed-Forward 3D Reconstruction and View Synthesis: A Survey"
type: paper
tags: [survey, feed-forward, 3d-reconstruction, novel-view-synthesis, NeRF, 3d-gaussian-splatting, pointmap, depth-estimation, SLAM]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf
url: https://arxiv.org/abs/2507.14501
status: draft
---

📄 [Full paper](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf) · [arXiv](https://arxiv.org/abs/2507.14501)

## TL;DR

This comprehensive survey covers the rapidly evolving field of feed-forward methods for 3D reconstruction and novel view synthesis -- learning-based models that infer 3D geometry or novel views in a single forward pass without per-scene optimization. The paper taxonomizes methods into five categories based on their underlying representation: NeRF-based, pointmap-based, 3D Gaussian Splatting-based, other 3D representations (mesh, SDF, occupancy), and 3D-free models. It reviews datasets, evaluation protocols, and applications spanning digital humans, SLAM, robotics, and image matching.

## Problem

Traditional 3D reconstruction methods (SfM, per-scene NeRF optimization, 3DGS optimization) require computationally intensive iterative optimization, limiting applicability in real-world scenarios. The explosion of feed-forward approaches since 2020 necessitates a systematic review to organize methods, identify trends, and highlight open challenges across this rapidly growing field.

## Method

The survey organizes the field into a five-category taxonomy:

### 1. NeRF-based Feed-Forward Models
- **1D feature methods**: CodeNeRF, Shap-E -- encode global latent codes
- **2D feature methods**: PixelNeRF, IBRNet, GNT -- project 3D points onto source views for features
- **3D volume features**: MVSNeRF, GeoNeRF, ENeRF -- construct cost volumes from multi-view features
- **3D triplane features**: LRM, Pf-LRM, TripoSR, Instant3D -- large transformer decoders regress triplane representations

### 2. Pointmap-based Models
- Pioneered by [[dust3r|DUSt3R]]: transformer encoder-decoder directly outputs pixel-aligned pointmaps from unposed, uncalibrated image pairs
- Extensions: [[mast3r|MASt3R]] (adds local features), Fast3R (global fusion transformer), Spann3R (spatial memory), CUT3R (continuous updating), [[vggt|VGGT]] (predicts all 3D attributes in one pass)
- Memory-based: Spann3R, MUSt3R, Point3R -- incrementally update scene representation
- SfM integration: Light3R-SfM, Regist3R
- Conditioning: [[Pow3R]] (camera/depth priors), Rig3R (rig metadata), MoGe (affine-invariant pointmaps)

### 3. 3DGS-based Models
- **Gaussian maps**: Splatter Image, PixelSplat, MVSplat, GS-LRM -- predict per-pixel Gaussian parameters
- **Gaussian volumes**: Triplane-based (GSDS, Triplane-GS) and volume-based approaches
- Epipolar-based: PixelSplat, LatentSplat
- Cost volume-based: MVSplat, MVSGaussian
- Pointmap-based: NoPoSplat, PreF3R

### 4. Other 3D Representations
- Mesh: MeshFormer, CRM, InstantMesh
- SDF/Occupancy: various approaches for implicit reconstruction

### 5. 3D-Free Models
- Image diffusion: Zero123, SV3D -- generate novel views without explicit 3D
- Video diffusion: ViewCrafter, Vivid-1-to-3
- Direct NVS without 3D inductive bias: LVSM

### Applications
- Multi-view reconstruction, dynamic scenes, large-scale reconstruction
- Image matching (DUSt3R, MASt3R used as matchers)
- SLAM (feed-forward methods for real-time tracking)
- Robotics (scene understanding from sparse views)
- Digital humans (avatar reconstruction)

## Results

The survey does not present novel experimental results but provides comprehensive comparison tables and analysis of existing benchmarks. Key observations:
- Feed-forward methods have achieved orders of magnitude faster inference than per-scene optimization
- Pointmap-based methods (DUSt3R family) have become dominant for unconstrained reconstruction
- 3DGS-based methods lead for real-time rendering quality
- The field has grown explosively: the timeline (Fig. 2) shows dozens of major works from 2021-2025

## Why it matters

This survey provides the first comprehensive taxonomy of the feed-forward 3D reconstruction field, which has grown from a handful of works (PixelNeRF, 2021) to a major research direction with hundreds of papers. It serves as an essential reference for understanding how methods relate to each other and where the field is heading. The clear categorization by representation type helps researchers identify which approach is most suitable for their specific application.

## Relation to prior work

The survey covers a vast network of related methods:
- Foundation works: [[nerf|NeRF]], [[3d-gaussian-splatting]], [[dust3r|DUSt3R]], [[mast3r|MASt3R]]
- Pointmap family: [[Pow3R]], [[Spann3R]], [[MonST3R]], [[CUT3R]], [[vggt|VGGT]], [[Fast3R]]
- Gaussian splatting: [[PixelSplat]], [[MVSplat]], Splatter Image
- Large reconstruction: [[LRM]], TripoSR
- Classical pipelines: [[colmap|COLMAP]], [[bundle-adjustment]], [[SfM]]
- Diffusion-based: Zero123, SV3D, RenderDiffusion
- Feature matching: DUSt3R and MASt3R used for [[feature-matching]]
- Pre-training: [[CroCo]], [[dinov2|DINOv2]] as backbones

## Open questions / limitations

- Limited modality diversity in training datasets (mostly RGB; few with LiDAR, thermal, event cameras)
- Poor generalization in free-viewpoint synthesis for unseen viewpoint ranges
- High computational cost of long-context processing (memory scales quadratically with sequence length)
- Most methods assume static scenes; dynamic scene handling remains challenging
- Evaluation metrics (PSNR, SSIM, LPIPS) may not capture perceptual quality adequately
- Societal impact concerns: potential misuse for deepfakes and privacy violations

## References added to the wiki

- [[feed-forward-3d-reconstruction]]
- [[nerf|NeRF]]
- [[3d-gaussian-splatting]]
- [[dust3r|DUSt3R]]
- [[mast3r|MASt3R]]
- [[pointmap-regression]]
- [[vggt|VGGT]]
- [[PixelSplat]]
- [[MVSplat]]
- [[LRM]]
- [[colmap|COLMAP]]
