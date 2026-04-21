---
title: Depth-induced symmetric reprojection scoring (truncated MSAC)
type: idea
source_paper: wiki/papers/yu2025_madpose.md
also_in: []

scope: drop-in
stages: [pose-estimation.robust-estimator-scoring]
collapses: []
splits_into: []
rewrites: {}

inputs: [pose-hypothesis, correspondence, depth-priors-with-solved-affine-corrections, optional-intrinsics]
outputs: [per-correspondence-truncated-reprojection-score]
assumptions: [upstream-solver-returns-α-β1-β2-with-pose, static-pair]
requires_upstream_property: [affine-corrected-relative-pose-solver-output]
requires_downstream_property: [msac-accepts-per-correspondence-scalar-scores]
learned_params: []
failure_modes: [unreliable-depth-priors-inflate-error-even-after-correction, symmetric-truncation-threshold-τr-tuning]

requires: [affine-corrected-minimal-relative-pose-solvers_yu2025]
unlocks: [hybrid-lo-msac-dual-modality-estimator_yu2025]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [pose-estimation, ransac, msac, sampson, depth-aware-scoring]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
Classical pose RANSAC scores each correspondence using the Sampson epipolar error — a first-order approximation to the distance of the point from the epipolar line implied by the hypothesized pose. Sampson is pose-only; it sees nothing about metric structure.

This drop-in replaces (or supplements) Sampson with a **symmetric reprojection error** using the pose hypothesis and the upstream solver's `(α, β_1, β_2)` corrections. Lifted 3D points are:

$$P_1 = K_1^{-1} [p_1, 1]^\top (d_1 + β_1), \quad P_2 = K_2^{-1} [p_2, 1]^\top α(d_2 + β_2)$$

Compute both forward and backward reprojection:

$$E_{r(1 \to 2)}(\Theta) = \|\Pi_2(RP_1 + t) - p_2\|^2, \quad E_{r(2 \to 1)}(\Theta) = \|\Pi_1(R^{-1}P_2 - R^{-1}t) - p_1\|^2$$

Both directions are needed because an asymmetric reprojection would give undue weight to correspondences whose depth prior is accurate on one side and bad on the other; the symmetric form folds the two into one score.

Per-correspondence MSAC contribution is the truncated sum:

$$\bar{E}_r(\Theta; p_1, p_2) = \min(E_{r(1 \to 2)}, τ_r) + \min(E_{r(2 \to 1)}, τ_r)$$

with `τ_r` tuned per dataset (paper uses sub-pixel thresholds at different AUC@° granularities). Truncation is the MSAC trick that prevents outlier correspondences from dominating the score.

The score is fully determined by `(R, t, α, β_1, β_2)`, all of which the upstream solver provides — no additional computation beyond two matrix-vector products and two projections per correspondence.

## Why it wins
Causal story: Sampson scoring makes RANSAC hypothesis ranking depth-blind, so even with a depth-aware solver the outer loop loses depth information. Depth-induced reprojection restores that information: hypotheses whose solved `(α, β_1, β_2)` align depths across views score well on correspondences where both depth priors are reliable, and score poorly otherwise — which is precisely what the hybrid estimator needs to tell reliable-prior correspondences from noisy-prior ones.

Isolating ablation (Table 7 of [[yu2025_madpose]], ScanNet-1500 shared-focal, SP+SG + DA-met):
- All hybrid: 18.35 / 37.54 / 57.58 AUC@5/10/20°.
- H solver + H LO, P score (Sampson only): 14.06 / 29.86 / 48.13 — **−8 AUC@10° from dropping the hybrid score**.
- H solver, D LO, D score (pure depth-only): 16.60 / 35.35 / 55.26 — weaker than combined.
- P solver + P LO, P score (classical): 15.86 / 30.27 / 45.57.

The hybrid score row is the largest single-ablation drop — dropping it hurts more than dropping the hybrid solver (−2 AUC@10°) or the hybrid LO step (−2 AUC@10°). Empirically, **hybrid scoring is the hinge** of the overall gain.

## Preconditions & compatibility
- **Upstream**: requires a solver that returns `(α, β_1, β_2)` jointly with pose. [[affine-corrected-minimal-relative-pose-solvers_yu2025]] is the canonical provider; Barath 2022 2pt+D and Ding 2024 3p3d/4p4d provide `α` only (not sufficient — the scoring as stated needs shifts too).
- **Downstream**: any MSAC loop that consumes scalar per-correspondence scores. Compatible with PoseLib, RansacLib, Hybrid LO-MSAC (Camposeco 2018). Can also be used inside a GC-RANSAC wrapper.
- **Threshold tuning**: `τ_r` is dataset-dependent; paper reports robustness to a range of thresholds, but cross-domain deployment should re-tune.

## Trade-offs vs. the decomposed pipeline
Not applicable — `drop-in` scope. Replaces a single scalar scoring function; no pipeline shape change.

## Open questions
- **Combining with Sampson**: the paper's hybrid estimator uses Sampson for the `I_3` inlier type (correspondences consistent with pose but not with depth). Could the scoring be unified — e.g., use `min(E_r, Sampson)` per correspondence with an adaptive weight — rather than partitioning inliers into types? Unresolved.
- **Learned re-weighting**: could a small network predict per-correspondence score weights from image appearance + depth confidence, replacing the fixed truncation `τ_r`? Not proposed.
- **Extension to multi-view**: how do you score a 3-view pose hypothesis with multi-view depth-induced reprojection? Needs the shift-consistency constraint that is an open problem (see `affine-corrected-minimal-relative-pose-solvers_yu2025`'s open questions).
