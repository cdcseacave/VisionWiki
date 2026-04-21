---
title: Single-view depth lifting + depth-constrained BA + forward-backward consistency (MP-SfM)
type: idea
source_paper: wiki/papers/pataki2025_mp-sfm.md
also_in: []

scope: topology-rewrite
stages: [sfm.next-view-registration, sfm.bundle-adjustment, sfm.registration-validation]
collapses: []
splits_into: []
rewrites: {replaces: [sfm.next-view-registration, sfm.bundle-adjustment], introduces: [sfm.mono-depth-lifted-registration, sfm.depth-constrained-ba, sfm.forward-backward-depth-consistency]}

inputs: [unordered-images, metric3dv2-depth-and-normal]
outputs: [sfm-reconstruction-on-low-overlap-captures]
assumptions: [mono-depth-prior-available, static-scene]
requires_upstream_property: [metric3dv2-or-equivalent-depth-normal]
requires_downstream_property: [ceres-BA-accepts-depth-residuals]
learned_params: []
failure_modes: [depth-prior-quality-bounds-reconstruction, heterogeneous-gpu-cpu-pipeline]

requires: []
unlocks: []
co_requires: [matcher-score-next-view-selection_pataki2025, depth-proportional-uncertainty-fusion_pataki2025, bilateral-normal-integration-with-uncertainty_pataki2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, mono-depth, depth-normal-prior, low-overlap]
created: 2026-04-18
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Five coupled sub-mechanisms that together reshape classical incremental SfM around monocular priors:

1. **Two-view initialization with mono-depth fallback** (§3.1). Follow COLMAP's image-pair ranking. If the top pair has enough inliers and parallax, run standard essential-matrix init. Otherwise lift feature points from image $a$ using its mono-depth $D_a$ and intrinsics $K_a$ to form 2D-3D matches with image $b$; estimate absolute pose $T_{ba}$ via PnP. This handles low-parallax captures where classical init cannot bootstrap.

2. **Per-view depth-map rescaling** (Eq. 1). For each image, rescale the mono-depth by the median ratio to already-triangulated 3D points: $D^*_i = D_i \cdot \text{median}_{j,k}(\hat{D}_i(X_k)/D_i(x_j))$. Required before lifting — up-to-scale priors would otherwise produce inconsistent 3D extents across views.

3. **Single-view 3D lifting for next-view registration** (§3.2). Lift features in registered images to 3D via the rescaled per-view depth. Candidate new views register via 2D-3D correspondences with these lifted points, bypassing the classical three-view-track requirement. Ranks candidates by summed matcher score (see [[matcher-score-next-view-selection_pataki2025]]).

4. **Depth-constrained BA with normal integration** (§3.3). Joint objective $C_{BA} + C_{reg} + C_{int}$: reprojection + 3D-point-to-depth-map tie + depth-prior-plus-bilateral-normal-integration with propagated covariance ([[bilateral-normal-integration-with-uncertainty_pataki2025]]). Hessian is not Schur-amenable (depth-normal coupling breaks block-diagonal assumption) → alternating block coordinate descent: $C_{reg} + C_{int}$ per image (GPU), $C_{BA} + C_{reg}$ across views (CPU Ceres). Uncertainties come from [[depth-proportional-uncertainty-fusion_pataki2025]].

5. **Forward-backward depth consistency filter** (§3.4, Eq. 6). Reproject each refined depth map into overlapping views; measure fraction of pixels where depths disagree beyond the propagated-uncertainty confidence band. If this ratio exceeds the expected occlusion fraction, de-register the offending view. Catches symmetry-induced ghost cameras without a learned classifier.

## Why it wins

Causal story: classical incremental SfM fails on low-overlap / low-parallax / symmetric captures because it needs three-view tracks for scale consistency, and its track-based validation cannot distinguish symmetry ghosts from real registrations. Mono-priors provide an alternative scale signal (depth) and a per-view geometric fingerprint (refined dense depth) that both relaxes the three-view requirement and enables dense consistency-based filtering.

Evidence:
- ETH3D minimal (0%) three-view overlap with MASt3R matching: AUC@1°/5°/20° = **34.9/67.2/81.7** vs MASt3R-SfM 20.1/39.7/52.2 and COLMAP 1.1/2.3/3.1 (Table 1).
- SMERF minimal overlap: **17.2/54.6/77.1** with MASt3R — MP-SfM is the only approach that reconstructs these at all (Table 2).
- RealEstate10k low-parallax: **35.5/81.9/91.5** vs MASt3R-SfM 33.4/80.8/91.2 — wins AUC@1° specifically in low-parallax (Table 3).
- Robustness to mono-depth choice: wins across Metric3D-v2, DepthPro, DepthAnything-v2 — ablation isolates depth-uncertainty awareness as critical (Table 4).

Ablation isolation (Table 5): removing depth refinement, depth regularization, or depth lifting each individually degrades results, confirming the sub-mechanisms contribute independently within the bundle.

## Preconditions & compatibility

- Requires a mono-depth + mono-normal estimator with (or patched-to-have) per-pixel uncertainty — see `co_requires`.
- Heterogeneous compute: GPU depth refinement + CPU Ceres BA.
- Less accurate than MASt3R-SfM at AUC@1° on object-centric scenes (Tanks & Temples) due to lack of foreground matches — MP-SfM targets scene-level reconstruction.
- Bet #010 in [[gpu-native-sfm]] explores fusing this with InstantSfM's GPU-native BA; an open frontier.

## Pipeline-shape implications

Topology-rewrite at `rewrites:`: removes [[sfm.next-view-registration]] and [[sfm.bundle-adjustment]], introduces [[sfm.mono-depth-lifted-registration]], [[sfm.depth-constrained-ba]], and [[sfm.forward-backward-depth-consistency]]. A thread adopting this must represent the three-node subgraph, not just a BA swap — and by `co_requires:` also adopt [[matcher-score-next-view-selection_pataki2025]], [[depth-proportional-uncertainty-fusion_pataki2025]], and [[bilateral-normal-integration-with-uncertainty_pataki2025]] as bundle-siblings.

## Trade-offs vs. the decomposed pipeline

What the classical decomposed pipeline offered that this fused variant loses:
- **Per-stage interpretability**: classical SfM lets you inspect pose, triangulation, and depth independently. MP-SfM's alternating BA couples them; a failure mode in one sub-objective propagates through iterations.
- **Graceful degradation without priors**: classical SfM with sparse features still produces *something* on easy scenes. MP-SfM without a mono-prior doesn't function at all — the priors are load-bearing, not optional.
- **Swap-ability of internal stages**: in classical SfM you can replace BA while keeping geometric verification. MP-SfM's depth-constrained BA assumes the lifted-registration + consistency filter are present.

In exchange you get robustness to the failure regimes (low-overlap, low-parallax, symmetry) that kill the decomposed pipeline.

## Open questions

- Can the GPU depth refinement + CPU BA split be collapsed onto GPU end-to-end (what Bet #010 explores)? The Schur-complement-incompatibility holds regardless of compute platform.
- How does the system degrade as mono-priors get worse (e.g., outdoor vegetation)? Paper acknowledges vegetation as a limitation but doesn't graph the degradation curve.
- Does the median-ratio depth rescaling (Eq. 1) stay stable as the reconstruction grows — or does a view with few triangulated landmarks get a biased rescaling and cascade errors?
