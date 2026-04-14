---
title: "MV-DeepSimNets: Multi-view Dense Image Matching with Similarity Learning and Geometry Priors"
type: paper
tags: [multi-view-stereo, dense-matching, similarity-learning, geometry-priors, aerial-imagery, satellite-imagery, photogrammetry]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/mvs-depth/chebbi_2025_multiview-dense-matching.pdf
url: https://arxiv.org/abs/2505.11264
status: draft
---

📄 [Full paper](../../papers/mvs-depth/chebbi_2025_multiview-dense-matching.pdf) · [arXiv](https://arxiv.org/abs/2505.11264)

## TL;DR

MV-DeepSimNets extends stereo similarity learning networks (DeepSimNets) to multi-view dense image matching without requiring multi-view retraining. The method trains feature extractors on epipolar image pairs, then at inference time leverages geometric priors (epipolar or homographic warping) to transform native multi-view images into rotation-aligned geometries, enabling plane-sweep-based multi-view similarity computation. The approach is integrated into the MicMac photogrammetry software and demonstrates strong generalization across aerial and satellite imagery with varying ground sampling distances.

## Problem

End-to-end multi-view stereo networks are memory-intensive (they must hold the entire cost volume) and struggle with generalization to unseen landscapes and acquisition geometries. Training multi-view networks also requires laborious multi-view ground truth datasets. Meanwhile, stereo regression networks are inherently limited to the epipolar case, requiring post-processing fusion for multi-view settings. There was a need for a multi-view matching framework that is generic, easily transferable, and handles arbitrary multi-view configurations without exhaustive retraining.

## Method

The core idea is to decouple training (on epipolar stereo pairs) from multi-view inference (using geometric priors):

1. **Epipolar similarity learning**: Three CNN feature backbones are trained (U-Net32, Attention U-Net, and the lightweight MS-AFF) using a triplet loss for representation learning and BCE loss for similarity learning on epipolar image pairs. An occlusion-aware loss term regularizes occluded regions. Iterative negative sampling converges toward the n-pair loss.

2. **Geometry-aware features**: At inference, images are warped to rotation-aligned geometries via either:
   - **Epipolar warping** $T_{epip}$: transforms stereo pairs so correspondences lie along rows
   - **Homographic warping** $T_{hom}$: transforms query images to the reference image geometry via a plane-to-plane homography

   Features are extracted in the aligned geometry and warped back: $F_{I_R} = T_R^{-1}[E_{2D}(I_{T_R})]$

3. **Multi-view plane sweeping**: For each depth hypothesis $Z$, query features are ortho-rectified to the reference geometry using known camera poses. Similarities are computed via cosine distance or learned MLP, then averaged across views.

4. **Multi-resolution pipeline**: Low-resolution depths are initialized with handcrafted NCC features; learned features intervene at higher resolutions where they are more effective. Final depths are regularized with [[SGM]].

## Results

- **In-distribution (Dublin, aerial, GSD ~4cm)**: MS-AFF consistently outperforms MC-CNN and standard NCC across all metrics. Adding views (2 to 3) improves reconstruction for all methods. Epipolar prior consistently outperforms homographic prior.
- **Out-of-distribution generalization (Le Mans 21cm, Montpellier satellite 32cm)**: MS-AFF maintains strong performance across unseen landscapes and GSDs. PSMNet shows significant performance degradation on satellite imagery. RAFT-Stereo generalizes well but smooths fine details.
- **Accuracy**: On Dublin 3-view with epipolar prior, MS-AFF achieves mean error of 0.15m, NMAD of 0.06, and D3xGSD of 89.3%.
- **MS-AFF** is 8-10x lighter than U-Net variants while achieving comparable or better accuracy.

## Why it matters

This work demonstrates that powerful multi-view dense matching can be achieved by training only on stereo pairs and leveraging geometric priors at inference time. This dramatically reduces training data requirements and enables deployment on large-scale aerial and satellite photogrammetry projects where multi-view training datasets are scarce. The integration into [[MicMac]] makes it immediately practical for operational photogrammetry pipelines.

## Relation to prior work

- Extends [[DeepSimNets]] (Chebbi et al., 2023) from stereo to multi-view
- Contrasts with end-to-end MVS networks like [[MVSNet]], [[TransMVSNet]], and [[PatchMatchNet]] that require multi-view training
- Compares against [[MC-CNN]], [[PSMNet]], and [[RAFT-Stereo]]
- Uses [[SGM]] (Semi-Global Matching) for cost volume regularization
- Related to [[plane-sweeping]] approaches for [[multi-view-stereo]]
- Connects to [[nerf|NeRF]] research direction for complementary occlusion handling

## Open questions / limitations

- Homographic warping introduces directional sensitivity: features trained on horizontal epipolar disparities struggle with vertical displacements unless a 90-degree rotation is applied
- MS-AFF has not been trained on low-GSD imagery; handcrafted features are still needed at coarse pyramid levels
- The MLP similarity function encodes epipolar-specific information, making cosine similarity preferable when homographic warping is used
- Error metrics based on pixel-level histograms may not capture semantic quality of reconstructed surfaces (e.g., building edges vs. noise)

## References added to the wiki

- [[MV-DeepSimNets]]
- [[DeepSimNets]]
- [[plane-sweeping]]
- [[SGM]]
- [[multi-view-stereo]]
- [[epipolar-geometry]]
- [[MicMac]]
