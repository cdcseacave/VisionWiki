---
title: "RoMa v2: Harder Better Faster Denser Feature Matching"
type: paper
tags: [feature-matching, dense-matching, correspondence, DINOv3, multi-view-transformer, refinement, pose-estimation]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/feature-matching/edstedt_2025_roma-v2.pdf
url: https://arxiv.org/abs/2511.15706
status: draft
---

📄 [Full paper](../../papers/feature-matching/edstedt_2025_roma-v2.pdf) · [arXiv](https://arxiv.org/abs/2511.15706)

## TL;DR

RoMa v2 is a dense feature matcher that significantly improves upon RoMa through a series of systematic improvements: upgrading to frozen DINOv3 features, replacing the Gaussian Process with a Multi-view Transformer for coarse matching, a decoupled two-stage matching-then-refinement training pipeline, custom CUDA kernels for memory-efficient refinement, a curated diverse training distribution, per-pixel error covariance prediction, and EMA to eliminate sub-pixel bias. RoMa v2 achieves state-of-the-art results across a wide range of benchmarks while being significantly faster than RoMa.

## Problem

Dense feature matching -- estimating correspondences for every pixel between two images -- remains challenging for many real-world scenarios. RoMa is robust to extreme appearance changes (due to frozen foundation model features) but lacks multi-view context in its matcher, has significant runtime and memory footprint, and struggles under extreme viewpoint changes. UFM is faster but fine-tunes its backbone (hurting generalization to extreme appearance changes) and lacks sub-pixel precision. No single method combined robustness, speed, and sub-pixel accuracy.

## Method

### Architecture: Two-stage matching-then-refinement

**Coarse Matcher** (Figure 4):
1. Frozen [[DINOv3]] ViT-L extracts features from both images (layers 11 and 17, concatenated to 2048-dim, projected to 768-dim)
2. Features processed by a ViT-B **Multi-view Transformer** with alternating frame-wise and global attention (12 blocks, 768 dim)
3. DPT head outputs coarse warps and confidence at stride 4
4. Key change: Replaced RoMa's Gaussian Process with single-headed attention + auxiliary NLL loss

**Matching Loss**: Combines three terms:
$$L_{\text{matcher}} = L_{\text{NLL}} + L_{\text{warp}}(r_\theta, p_{GT}) + 10^{-2} L_{\text{overlap}}(p_\theta, p_{GT})$$

Where $L_{\text{NLL}}$ is a dense directional matching loss computed from the similarity matrix $S \in \mathbb{R}^{M \times N}$ between all patch pairs.

**Refiners** (Figure 5):
- Three CNN-based refiners at strides {4, 2, 1} using VGG19 features
- Local correlation via custom CUDA kernel (significantly reduces memory vs. RoMa's implementation)
- Channel dimensions set to powers of two for speed
- Predict warp displacement, confidence delta, and per-pixel 2x2 precision matrix

**Predictive Covariance**: Each refiner predicts Cholesky factors of a 2x2 precision matrix $\Sigma^{-1}$ via constrained parameterization (Softplus + lower triangular), trained with negative log-likelihood loss.

**EMA for bias removal**: Training exhibits random sub-pixel bias (~0.1px) that fluctuates over iterations. An exponential moving average with decay 0.999 eliminates this bias.

### Training Data
Curated mixture of wide-baseline (MegaDepth, AerialMegaDepth, TartanAir V2, MapFree, BlendedMVS) and small-baseline (ScanNet++ v2, Hypersim, FlyingThings3D, UnrealStereo4K, Virtual KITTI 2) datasets. Matcher trained 300k steps (batch 128), refiners trained 300k steps (batch 64).

## Results

- **MegaDepth-1500**: State-of-the-art pose estimation AUC, outperforming both RoMa and UFM
- **ScanNet-1500**: Strong indoor performance
- **WxBS (extreme appearance changes)**: Maintains RoMa's robustness advantage over UFM
- **ScanNet++ v2**: Excellent on high-fidelity indoor scenes
- **TartanAir V2, SatAst (new benchmark), FlyingThings3D, MapFree, AerialMegaDepth**: Consistent top performance across all benchmarks (see radar chart, Figure 1)
- **Hypersim dense matching**: RoMa v2 matcher significantly outperforms UFM (PCK@3px: 30.5% vs 11.2%)
- **Runtime**: Significantly faster than RoMa due to stride-4 matching (vs stride-14) requiring fewer refinement stages
- **Memory**: Custom CUDA kernel substantially reduces refinement memory (Table 8)
- **Predictive covariance**: Improves downstream geometry estimation when used as weights in Sampson error optimization

## Why it matters

RoMa v2 advances the state of the art in dense feature matching by resolving the tension between robustness (frozen foundation features), accuracy (sub-pixel precision with covariance), and efficiency (decoupled training, CUDA kernels). Dense matching is foundational for visual localization, 3D reconstruction, and SLAM, so improvements here propagate to many downstream tasks. The predictive covariance output is particularly valuable for robust geometry estimation.

## Relation to prior work

- Builds on [[RoMa]] (Edstedt et al., CVPR 2024) and addresses its limitations
- Competes with [[UFM]] (Zhang et al., NeurIPS 2025) -- RoMa v2 combines UFM's speed advantages with RoMa's robustness
- Upgrades from [[DINOv2]] to [[DINOv3]] as frozen feature backbone
- Architecture inspired by [[dust3r|DUSt3R]], [[mast3r|MASt3R]], and [[vggt|VGGT]] for multi-view transformer design
- Follows alternating attention pattern from [[vggt|VGGT]] (Wang et al.)
- Uses DPT head from [[DPT]] (Ranftl et al., 2021)
- Relates to sparse matchers [[SuperGlue]], [[LightGlue]] and semi-dense [[LoFTR]]
- Coarse matching relates to [[DKM]] (predecessor of RoMa)
- Introduces SatAst benchmark using astronaut/satellite image pairs from EarthMatch

## Open questions / limitations

- Some spurious confidence in textureless sky regions, possibly due to AerialMegaDepth depth leaking into sky pixels
- DINOv3 has slightly larger patch size (16 vs 14) than DINOv2, which may limit finest-scale matching
- The decoupled two-stage training prevents end-to-end gradient flow between matcher and refiners
- Predictive covariance training assumes Gaussian error distribution, which may not hold for all failure modes
- New SatAst benchmark is small (39 pairs) and limited to homography-related scenes

## References added to the wiki

- [[RoMa-v2]]
- [[RoMa]]
- [[DINOv3]]
- [[dense-feature-matching]]
- [[multi-view-transformer]]
- [[DPT]]
- [[UFM]]
- [[SuperGlue]]
- [[LightGlue]]
- [[LoFTR]]
