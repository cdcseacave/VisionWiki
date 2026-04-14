---
title: "MP-SfM: Monocular Surface Priors for Robust Structure-from-Motion"
type: paper
tags: [sfm, monocular-depth, surface-normals, incremental-sfm, low-overlap, robustness]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/pataki_2025_mp-sfm.pdf
url: https://arxiv.org/abs/2504.20040
status: draft
---

📄 [Full paper](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf) · [arXiv](https://arxiv.org/abs/2504.20040)

## TL;DR

MP-SfM augments the classical incremental [[structure-from-motion]] paradigm (building on [[colmap|COLMAP]]) with monocular depth and surface normal priors from deep networks, enabling accurate multi-view reconstruction from only two-view tracks. This lifts the long-standing requirement for three-view overlap, making SfM robust under extreme viewpoint changes, low parallax, and symmetry-induced failures while maintaining performance in standard conditions.

## Problem

State-of-the-art SfM systems fundamentally require three-view overlap with sufficient baseline and parallax to perform multi-view consistent 3D reconstruction. This requirement is difficult to satisfy in practice--especially for non-expert users capturing complex scenes--leading to frequent failures under low overlap, low parallax, or repetitive structures/symmetries. Dense matchers like [[mast3r|MASt3R]] can find correspondences across extreme viewpoints, but feeding these into existing reconstruction algorithms still fails.

## Method

MP-SfM integrates monocular depth and normal priors into incremental SfM through several key mechanisms:

1. **Single-view 3D point lifting**: Feature points are lifted to 3D using monocular depth, enabling next-view registration from only two-view correspondences (no three-view tracks required). This allows leveraging dense pairwise matchers directly.

2. **Depth-constrained bundle adjustment**: The optimization alternates between (a) depth refinement via normal integration with uncertainty weighting, and (b) joint BA over poses, 3D points, and refined depth maps. Principled uncertainty propagation from monocular priors ensures robustness to prediction errors.

3. **Dense depth consistency check**: Reprojected depth maps across views are compared to identify incorrect registrations (e.g., from symmetries), using a forward-backward consistency metric with occlusion handling.

The objective decomposes into three terms: standard reprojection ($C_{BA}$), depth regularization ($C_{reg}$), and depth integration with normal constraints ($C_{int}$), solved via alternating block coordinate descent.

## Results

- **ETH3D (low overlap)**: At minimal three-view overlap (0%), AUC@1/5/20° of 34.9/67.2/81.7 with MASt3R matching vs. 20.1/39.7/52.2 for MASt3R-SfM and 8.4/15.8/22.5 for GLOMAP.
- **SMERF**: Only approach capable of reconstructing minimal-overlap scenes (AUC 17.2/54.6/77.1 with MASt3R matching).
- **Tanks & Temples**: Competitive across all overlap levels; best robustness (AUC@20°) in low-overlap settings.
- **RealEstate10k (low parallax)**: AUC@1/10/30° of 35.5/81.9/91.5 with MASt3R, outperforming MASt3R-SfM (33.4/80.8/91.2) and COLMAP variants.
- Robust across different monocular depth models (Metric3D-v2, DepthPro, DepthAnything-v2); depth uncertainty is critical for fusion quality.

## Why it matters

MP-SfM demonstrates that integrating monocular priors into classical incremental SfM can address its most fundamental limitation (three-view overlap requirement) while retaining scalability and generality. As [[monocular-depth-estimation]] models continue to improve, this approach will automatically benefit with little tuning. It represents a pragmatic middle ground between fully classical and fully learned SfM.

## Relation to prior work

- Built on [[colmap|COLMAP]]'s incremental SfM framework with significant modifications to initialization, registration, refinement, and filtering.
- Uses [[mast3r|MASt3R]] or RoMa for dense correspondences; SuperPoint + LightGlue for sparse features.
- Contrasts with [[mast3r|MASt3R]]-SfM, which uses a global SfM paradigm and embeds some monocular priors implicitly but struggles more in low-overlap scenarios.
- Related to StudioSfM (low-parallax depth integration) but more general; handles unstructured image collections.
- Extends [[dust3r|DUSt3R]]/[[mast3r|MASt3R]] two-view reconstruction priors into a full multi-view pipeline.
- Uses [[Metric3Dv2]] for depth and normals, with [[DSINE]] as an alternative normal estimator.

## Open questions / limitations

- Less accurate than MASt3R-SfM at AUC@1° in object-centric scenes (Tanks & Temples) due to lack of foreground matches.
- Incremental SfM paradigm retains sequential processing overhead; does not parallelize as well as global methods.
- Depth integration optimization runs on GPU while BA uses Ceres on CPU, creating a heterogeneous compute pipeline.

## References added to the wiki

- [[structure-from-motion]]
- [[colmap|COLMAP]]
- [[GLOMAP]]
- [[mast3r|MASt3R]]
- [[dust3r|DUSt3R]]
- [[monocular-depth-estimation]]
- [[Metric3Dv2]]
- [[DSINE]]
- [[bundle-adjustment]]
- [[SuperPoint]]
- [[LightGlue]]
