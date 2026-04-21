---
title: "MP-SfM: Monocular Surface Priors for Robust Structure-from-Motion"
type: paper
tags: [sfm, monocular-depth, surface-normals, incremental-sfm, low-overlap, robustness]
created: 2026-04-12
updated: 2026-04-21
sources: []
local_paper: papers/sfm-slam/pataki_2025_mp-sfm.pdf
url: https://arxiv.org/abs/2504.20040
code: https://github.com/cvg/mpsfm
license_code: Apache-2.0
license_paper: arxiv-nonexclusive
status: draft
---

📄 [Full paper](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf) · [arXiv](https://arxiv.org/abs/2504.20040) · [code](https://github.com/cvg/mpsfm)

_Code license: `Apache-2.0`_

## TL;DR

MP-SfM augments the classical incremental [[structure-from-motion]] paradigm (building on [[colmap|COLMAP]]) with monocular depth and surface normal priors from deep networks, enabling accurate multi-view reconstruction from only two-view tracks. This lifts the long-standing requirement for three-view overlap, making SfM robust under extreme viewpoint changes, low parallax, and symmetry-induced failures while maintaining performance in standard conditions.

## Problem

State-of-the-art SfM systems fundamentally require three-view overlap with sufficient baseline and parallax to perform multi-view consistent 3D reconstruction. This requirement is difficult to satisfy in practice--especially for non-expert users capturing complex scenes--leading to frequent failures under low overlap, low parallax, or repetitive structures/symmetries. Dense matchers like [[mast3r|MASt3R]] can find correspondences across extreme viewpoints, but feeding these into existing reconstruction algorithms still fails.

## Method

MP-SfM integrates monocular depth and normal priors into incremental SfM through five coupled mechanisms:

1. **Two-view initialization with mono-depth fallback** (§3.1): When the top-ranked image pair lacks the inliers/parallax for essential-matrix init, lift points from image $a$ via $D_a$ and $K_a$ to form 2D-3D matches with image $b$; estimate pose $T_{ba}$ via PnP. Enables bootstrapping on low-parallax captures.

2. **Per-view depth rescaling** (Eq. 1): Rescale each mono-depth by the median ratio to already-triangulated 3D points, $D^*_i = D_i \cdot \text{median}_{j,k}(\hat{D}_i(X_k)/D_i(x_j))$, before lifting — absorbs the up-to-scale ambiguity of mono-priors.

3. **Single-view 3D point lifting for next-view registration**: Lifted features are used as 2D-3D correspondences for PnP, enabling registration from only two-view tracks. Next-view candidates are ranked by summed matcher score, not inlier count.

4. **Depth-constrained BA with normal integration**: Joint $C_{BA} + C_{reg} + C_{int}$ objective — reprojection, 3D-point-to-refined-depth tie, and depth-prior + bilateral-normal-integration with propagated covariance. Hessian is not Schur-amenable, so alternate block coordinate descent: $C_{reg} + C_{int}$ per image on GPU, $C_{BA} + C_{reg}$ across views on CPU (Ceres).

5. **Forward-backward depth consistency check**: Reprojected refined depths across views identify incorrect registrations (symmetry ghosts), using a propagated-uncertainty confidence band and occlusion-ratio threshold.

The system also **calibrates monocular-prior uncertainties**: take the pixel-wise max of model-predicted uncertainty and a depth-proportional uncertainty, clipped at 2 cm. Calibration scale tuned once on ETH3D training split.

## Results

- **ETH3D (low overlap)**: At minimal three-view overlap (0%), AUC@1/5/20° of 34.9/67.2/81.7 with MASt3R matching vs. 20.1/39.7/52.2 for MASt3R-SfM and 8.4/15.8/22.5 for GLOMAP.
- **SMERF**: Only approach capable of reconstructing minimal-overlap scenes (AUC 17.2/54.6/77.1 with MASt3R matching).
- **Tanks & Temples**: Competitive across all overlap levels; best robustness (AUC@20°) in low-overlap settings.
- **RealEstate10k (low parallax)**: AUC@1/10/30° of 35.5/81.9/91.5 with MASt3R, outperforming MASt3R-SfM (33.4/80.8/91.2) and COLMAP variants.
- Robust across different monocular depth models (Metric3D-v2, DepthPro, DepthAnything-v2); depth uncertainty is critical for fusion quality.

## Why it matters

MP-SfM demonstrates that integrating monocular priors into classical incremental SfM can address its most fundamental limitation (three-view overlap requirement) while retaining scalability and generality. As [[monocular-depth-estimation]] models continue to improve, this approach will automatically benefit with little tuning. It represents a pragmatic middle ground between fully classical and fully learned SfM.

## Pipeline contribution

- [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] — system-level `topology-rewrite`. Candidate thread: [[feed-forward-structure-from-motion]] Tier 2 · replaces: [[sfm.next-view-registration]] + [[sfm.bundle-adjustment]] · introduces: [[sfm.mono-depth-lifted-registration]], [[sfm.depth-constrained-ba]], [[sfm.forward-backward-depth-consistency]] · expected gain: AUC@1° 34.9 vs GLOMAP 8.4 on ETH3D 0%-overlap.
- [[matcher-score-next-view-selection_pataki2025]] — `stage-swap` on [[sfm.next-view-registration]] / [[sfm.next-view-scheduling]]. Candidate thread: [[feed-forward-structure-from-motion]] Tier 2 · reusable independently for any incremental SfM consuming a deep matcher (MASt3R / RoMa / LightGlue) · expected gain: avoids catastrophic failure on symmetric scenes with MASt3R-class matchers (Table 8).
- [[depth-proportional-uncertainty-fusion_pataki2025]] — `drop-in` recipe for calibrating off-the-shelf mono-depth uncertainties. Candidate thread: [[mono-depth-estimation]] · reusable across classical SfM, 3DGS depth supervision, MVS · expected gain: recovers calibrated uncertainty for consumers that inverse-variance-weight depth priors.
- [[bilateral-normal-integration-with-uncertainty_pataki2025]] — `stage-swap` sub-mechanism inside [[sfm.depth-constrained-ba]]. Propagates normal uncertainty through the integration residual. Reusable for any joint-depth-pose refinement with normal priors.
- **Synthesis-bet candidate** (extends existing Bet #010 in [[gpu-native-sfm]]): Port [[depth-proportional-uncertainty-fusion_pataki2025]] and [[bilateral-normal-integration-with-uncertainty_pataki2025]] into [[depth-constrained-jacobian_zhong2026]]'s GPU-native BA — InstantSfM uses Metric3Dv2 depth but skips both the uncertainty calibration and the normal-integration term, leaving accuracy on the table.

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
- [[glomap|GLOMAP]]
- [[mast3r|MASt3R]]
- [[dust3r|DUSt3R]]
- [[monocular-depth-estimation]]
- [[Metric3Dv2]]
- [[DSINE]]
- [[bundle-adjustment]]
- [[SuperPoint]]
- [[LightGlue]]
