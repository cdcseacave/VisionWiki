---
title: "Pow3R: Empowering Unconstrained 3D Reconstruction with Camera and Scene Priors"
type: paper
tags: [3d-reconstruction, depth-estimation, pointmap-regression, foundation-model, depth-completion, multi-view-stereo, pose-estimation]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mvs-depth/jang_2025_pow3r.pdf
url: https://arxiv.org/abs/2503.17316
status: draft
---

📄 [Full paper](../../papers/mvs-depth/jang_2025_pow3r.pdf) · [arXiv](https://arxiv.org/abs/2503.17316)

## TL;DR

Pow3R extends the [[dust3r|DUSt3R]] framework by enabling the injection of any subset of auxiliary information -- camera intrinsics, relative poses, and sparse/dense depth maps -- into the feed-forward 3D reconstruction pipeline. By conditioning the ViT encoder and decoder on these optional modalities through lightweight MLPs, Pow3R performs on par with DUSt3R when no priors are available but substantially outperforms it when auxiliary information exists. It also gains new capabilities like native high-resolution processing via sliding window inference and depth completion.

## Problem

[[dust3r|DUSt3R]] and [[mast3r|MASt3R]] can only take RGB images as input. In many real-world applications, additional information is available (calibrated intrinsics, LiDAR sparse depth, known relative poses from IMU), yet these models have no mechanism to exploit it. This leaves significant performance on the table and prevents capabilities like processing images at native resolution or completing sparse depth maps.

## Method

Built on the [[dust3r|DUSt3R]] architecture (shared ViT encoder + two cross-attention decoders):

1. **Three pointmap outputs**: Unlike DUSt3R which outputs $X^{1,1}$ and $X^{2,1}$, Pow3R also predicts $X^{2,2}$ (pointmap of image 2 in its own coordinate system), enabling single-pass recovery of both cameras via Procrustes alignment.

2. **Versatile conditioning** via dedicated injection modules:
   - **Intrinsics**: Encoded as dense camera rays $K^{-1}[i,j,1]$, patchified and embedded, then added token-wise in encoder blocks via per-block MLPs ('inject-1' strategy)
   - **Depth/point clouds**: Normalized depth $D' = D/\text{norm}(D)$ and sparsity mask $M$ are patchified as $[D', M]$ and injected into encoder
   - **Relative pose**: $P_{12} = [R_{12}|t_{12}]$ with normalized translation is embedded and added to the CLS token in decoder blocks

3. **Random modality dropout during training**: Random subsets of modalities are provided at each iteration, enabling the model to handle any combination at test time.

4. **Confidence-aware loss**: Scale-invariant regression loss weighted by predicted per-pixel confidence: $L_{\text{conf}} = \sum C_{i,j} L_{i,j}^{\text{regr}} - \alpha \log C_{i,j}$ with $\alpha = 0.2$.

5. **High-resolution processing**: Camera intrinsics input encodes crop position, enabling sliding window inference at native resolution -- a key advantage over DUSt3R.

## Results

- **Depth estimation (zero-shot)**: On NYUd, Pow3R with all priors achieves significantly better results than DUSt3R, with improvements growing as more auxiliary information is provided. The depth completion capability outperforms specialized methods like CompletionFormer across varying sparsity levels.
- **Multi-view depth (DTU)**: Pow3R with pose and intrinsics outperforms DUSt3R and most classical/learning-based MVS approaches including [[colmap|COLMAP]].
- **Pose estimation (MegaDepth)**: Single-pass Procrustes-based pose recovery (from $X^{2,1}$ and $X^{2,2}$) is orders of magnitude faster than DUSt3R's PnP approach while being more accurate.
- **High-resolution (KITTI)**: Coarse-to-fine sliding window approach handles KITTI's unusual aspect ratio (370x1226) in zero-shot setting, producing detailed depth maps.
- **Controllability**: The model gracefully degrades when auxiliary information deviates from ground truth, and outputs low confidence for clearly incorrect inputs (e.g., focal $0.1 f_{gt}$).

## Why it matters

Pow3R demonstrates that feed-forward 3D reconstruction models can be made significantly more practical by accepting optional auxiliary information in a single unified architecture. This bridges the gap between "use what you have" practical scenarios and the RGB-only paradigm of [[dust3r|DUSt3R]]/[[mast3r|MASt3R]]. The approach is complementary to all DUSt3R-derived works (Spann3R, MonST3R, Splatt3R, etc.) and could improve them all.

## Pipeline contribution

- **Versatile conditioning (intrinsics / poses / depth) via per-block MLP injection (N1)** — any subset of auxiliary priors fed into DUSt3R's ViT encoder/decoder. candidate thread: [[feed-forward-structure-from-motion]] Tier 2 · stage: *prior injection into feed-forward backbone* · replaces/augments: *RGB-only DUSt3R / MASt3R* · expected gain: substantially beats DUSt3R when priors exist (MVS DTU); parity when they don't.
- **Random modality dropout training (N2)** — enables arbitrary combinations at inference. candidate thread: *feed-forward prior injection* · stage: *training recipe* · expected gain: one model for all prior combinations; no per-setup fine-tuning.
- **Third pointmap output $X^{2,2}$ (N3)** — Procrustes alignment recovers both cameras in single pass. candidate thread: [[feed-forward-structure-from-motion]] Tier 2/3 · stage: *pose recovery* · replaces/augments: *PnP on single pointmap* · expected gain: orders of magnitude faster pose recovery than DUSt3R's PnP loop.
- **Sliding-window high-res inference via crop-position encoding (N4)** — intrinsics input conveys crop location. candidate thread: *feed-forward inference* · stage: *resolution handling* · expected gain: native-resolution outputs without retraining.
- **Synthesis-bet candidate**: *inject RoMa v2 covariance-weighted sparse depth* from [edstedt2025_roma-v2] *as Pow3R's depth prior* — no paper does this; the matcher+pointmap combination should strictly improve DTU numbers.

## Relation to prior work

- Builds directly on [[dust3r|DUSt3R]] and is complementary to [[mast3r|MASt3R]]
- Related to [[Spann3R]], [[MonST3R]], [[Splatt3R]] which extend DUSt3R in other directions
- Depth completion relates to [[CompletionFormer]] and CSPN-family methods
- Focal estimation connects to [[UniDepth]] which also optionally takes intrinsics
- Camera ray representation inspired by [[RayDiffusion]] (Zhang et al., 2024)
- Uses [[CroCo]] v2 pre-training for the ViT backbone
- Contrasts with [[colmap|COLMAP]] which is optimization-based and cannot leverage learned priors

## Open questions / limitations

- Training is limited to image pairs; multi-view extension (N>2) would require architectural changes
- High-resolution sliding window approach assumes intrinsics are known; without them, DUSt3R's resolution limitation persists
- The model does not handle dynamic scenes (unlike MonST3R)
- Scale ambiguity remains when no depth priors are provided (as in DUSt3R)
- Coarse-to-fine high-resolution strategy requires solving for scale in overlapping crop regions

## References added to the wiki

- [[Pow3R]]
- [[dust3r|DUSt3R]]
- [[mast3r|MASt3R]]
- [[pointmap-regression]]
- [[depth-completion]]
- [[CroCo]]
