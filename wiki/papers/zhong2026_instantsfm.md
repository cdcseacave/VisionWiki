---
title: "InstantSfM: Towards GPU-Native SfM for the Deep Learning Era"
type: paper
tags: [sfm, gpu-acceleration, bundle-adjustment, global-sfm, pytorch, depth-priors, metric-scale]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/sfm-slam/zhong_2026_instantsfm.pdf
url: https://arxiv.org/abs/2510.13310
code: https://github.com/cre185/InstantSfM
license_code: NOASSERTION (custom)
license_paper: CC-BY-NC-SA-4.0
status: draft
---

📄 [Full paper](../../papers/sfm-slam/zhong_2026_instantsfm.pdf) · [arXiv](https://arxiv.org/abs/2510.13310) · [code](https://github.com/cre185/InstantSfM)

_Code license: `NOASSERTION (custom)`_

## TL;DR

InstantSfM is a fully GPU-based, PyTorch-compatible global [[structure-from-motion]] system that embeds metric depth priors directly into both global positioning and [[bundle-adjustment]] through a depth-constrained Jacobian structure, and introduces dynamic parameter extraction for robust outlier handling. It achieves up to 40x speedup over [[colmap|COLMAP]] and 12x over [[glomap|GLOMAP]] while maintaining competitive reconstruction accuracy.

## Problem

Mature SfM systems like [[colmap|COLMAP]] and [[glomap|GLOMAP]] remain CPU-centric and built on traditional C++ optimization toolchains, creating a growing mismatch with modern GPU-based, learning-driven pipelines (e.g., [[3d-gaussian-splatting]], [[nerf|NeRF]]). Existing GPU-accelerated [[bundle-adjustment]] frameworks are limited to reprojection-only objectives and cannot incorporate heterogeneous constraints like metric depth. Additionally, aggressive outlier filtering can leave cameras or 3D points under-constrained, causing rank-deficient normal equations and solver failure.

## Method

InstantSfM contributes two core technical ideas:

1. **Depth-Constrained Jacobian Structure**: Metric depth priors (from RGB-D sensors or [[monocular-depth-estimation]]) are embedded directly into global positioning (GP) and BA. In GP, per-observation scale variables are pinned to known depth values, propagating metric scale through shared camera-center columns in the Jacobian. In BA, an additional depth residual term is added with a binary validity mask for handling incomplete depth maps uniformly on GPU.

2. **Dynamic Parameter Extraction**: At each [[Levenberg-Marquardt]] iteration, the system identifies geometrically valid observations, extracts only active cameras and points into a compact parameter space, and remaps indices. This guarantees the Jacobian has no all-zero columns, maintaining full-rank normal equations even when large fractions of points are temporarily invalid.

The entire pipeline is implemented in PyTorch using sparse-aware GPU operations, supporting both GP and BA in a unified framework.

## Results

- **MipNeRF360**: Best average NVS metrics (PSNR 28.43, SSIM 0.856, LPIPS 0.113) across 7 scenes compared to [[colmap|COLMAP]], [[glomap|GLOMAP]], and [[VGGSfM]].
- **DTU**: Average PSNR 24.57 vs GLOMAP's 17.98 (GLOMAP fails on multiple scenes).
- **ScanNet**: Both COLMAP and GLOMAP fail on most scenes; InstantSfM succeeds on all, achieving Chamfer distance 0.658 with depth priors.
- **ScanNet++**: Average Chamfer distance 2.61 vs GLOMAP's 3.80.
- **Runtime**: 1.5x--40x faster than COLMAP, up to 12x faster than GLOMAP on scenes from 100 to 5000 images. GPU-accelerated COLMAP/GLOMAP variants still lag behind.

## Why it matters

InstantSfM represents the first fully PyTorch-native global SfM system, enabling seamless integration with modern learning pipelines for [[3d-gaussian-splatting]], [[nerf|NeRF]], and robotics. The depth-constrained formulation provides a principled way to fuse metric depth into SfM optimization rather than treating it as a post-processing alignment step, which is increasingly relevant as [[monocular-depth-estimation]] models improve.

See [[gpu-native-sfm]] thread for the broader "Tier 1: accelerated classical SfM" story this paper anchors.

## Pipeline contribution

- **Depth-constrained Jacobian structure (N1)** — mono-depth priors enter both global positioning and BA through shared camera-center Jacobian columns, not as post-hoc alignment. candidate thread: [[gpu-native-sfm]] · stage: *global positioning + BA* · replaces/augments: *reprojection-only Jacobians* · expected gain: metric scale preserved end-to-end; depth priors as first-class residuals alongside reprojection.
- **Dynamic parameter extraction (N2)** — per-LM-iteration identification of geometrically valid cameras/points, compact reparameterization to guarantee full-rank normal equations. candidate thread: [[gpu-native-sfm]] · stage: *LM solver* · replaces/augments: *damping hacks for rank deficiency* · expected gain: no solver failures when large outlier fractions temporarily invalidate points.
- **Fully PyTorch-native pipeline (N3)** — sparse-aware GPU operations for both GP and BA. candidate thread: [[gpu-native-sfm]] · stage: *implementation* · expected gain: 1.5×–40× over COLMAP; 12× over GLOMAP; first SfM system that lives inside a deep-learning graph.
- **Synthesis-bet enablement**: PyTorch-native design is the prerequisite for the [[gpu-native-sfm]] synthesis bet *"backprop rendering loss through SfM poses"* — differentiable joint SfM + 3DGS training is trivially implementable on this stack; no prior SfM allowed it.

## Relation to prior work

- Extends GPU-accelerated sparse [[bundle-adjustment]] from eager-mode BA (Zhan et al., 2024) into a complete global SfM system.
- Directly competes with [[colmap|COLMAP]] (incremental SfM) and [[glomap|GLOMAP]] (global SfM), offering a GPU-native alternative.
- Builds on [[rotation-averaging]] and global positioning formulations from GLOMAP.
- Contrasts with learning-based SfM like [[VGGSfM]] and feed-forward methods like [[dust3r|DUSt3R]], which are limited in scale or generalization.

## Open questions / limitations

- Designed for single-node execution; distributed settings not yet supported.
- Robustness under more challenging imaging conditions (e.g., extreme lighting, heavy occlusion) not extensively evaluated.
- Relies on external feature extraction and matching (RootSIFT); not an end-to-end learned system.

## Code & license

GitHub reports NOASSERTION — LICENSE file does not map to a standard SPDX ID. Verify terms before commercial redistribution.

## References added to the wiki

- [[structure-from-motion]]
- [[bundle-adjustment]]
- [[colmap|COLMAP]]
- [[glomap|GLOMAP]]
- [[VGGSfM]]
- [[monocular-depth-estimation]]
- [[3d-gaussian-splatting]]
- [[Levenberg-Marquardt]]
- [[rotation-averaging]]
- [[dust3r|DUSt3R]]
