---
title: Hybrid LO-MSAC with dual-modality solver alternation (MADPose)
type: idea
source_paper: wiki/papers/yu2025_madpose.md
also_in: []

scope: stage-swap
stages: [pose-estimation.hybrid-robust-estimator]
collapses: []
splits_into: []
rewrites: {}

inputs: [correspondences, per-view-mono-depth-priors, point-based-solver, depth-aware-solver, sampson-score, depth-reprojection-score, optional-intrinsics]
outputs: [final-relative-pose, affine-corrections, optional-focal-lengths, inlier-partition-by-data-type]
assumptions: [mono-depth-available-but-possibly-unreliable, static-pair, at-least-one-modality-has-inliers]
requires_upstream_property: [both-solvers-return-pose-and-optional-affine-corrections]
requires_downstream_property: [downstream-accepts-pose-plus-inlier-mask-plus-inlier-type-labels]
learned_params: []
failure_modes: [both-modalities-catastrophically-fail-on-degenerate-geometry, λ_s-and-τ-tuning-cross-domain, symmetric-scene-inlier-partition-biased]

requires: [affine-corrected-minimal-relative-pose-solvers_yu2025, depth-induced-reprojection-scoring_yu2025]
unlocks: []
co_requires: [affine-corrected-minimal-relative-pose-solvers_yu2025, depth-induced-reprojection-scoring_yu2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [pose-estimation, lo-ransac, lo-msac, hybrid, multi-modal, graceful-degradation]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
The MADPose hybrid estimator builds on the LO-RANSAC framework of Chum 2003, the fix-up of Lebeda 2012 ("LO-MSAC"), and the hybrid-solver idea of Camposeco 2018. Per RANSAC iteration:

1. **Solver choice**. Two solvers are present: (a) [[affine-corrected-minimal-relative-pose-solvers_yu2025]] (depth-aware), (b) classical point-based (Nistér 5pt calibrated / Stewénius 6pt shared-focal / 7pt fundamental two-focal). Initial selection probability is `0.5`. Online, probabilities are updated from inlier-type ratios: if depth-aware-solver hypotheses produce more inliers on data types `I_1, I_2` (see below), its probability rises, and vice versa.

2. **Minimal sample** from the full correspondence set, **hypothesize** via the selected solver (returns up to 4 or 8 pose+correction candidates), **score** each candidate.

3. **Data types + scoring**. Each correspondence is evaluated against three scores:
   - `E_{r(1→2)}` — forward depth-induced reprojection ([[depth-induced-reprojection-scoring_yu2025]]).
   - `E_{r(2→1)}` — backward depth-induced reprojection.
   - `E_s` — Sampson epipolar error (classical, pose-only).
   Inlier types:
   - `I_1` = `{i : E_{r(1→2),i} < τ_r}` (depth in view 1 consistent with pose + corrections).
   - `I_2` = `{i : E_{r(2→1),i} < τ_r}` (depth in view 2 consistent).
   - `I_3` = `{i : E_{s,i} < τ_s}` (epipolar-only inliers — includes points where one or both depth priors are wrong).
   A correspondence can be in multiple types. Hybrid MSAC score:

$$\bar{E}(\Theta; p_1, p_2) = \bar{E}_r + 2λ_s \frac{τ_r}{τ_s} \bar{E}_s, \quad \bar{E}_r = \min(E_{r(1→2)}, τ_r) + \min(E_{r(2→1)}, τ_r), \quad \bar{E}_s = \min(\min(E_s, τ_s), τ_r)$$

with `λ_s = 1` empirically. The `τ_r / τ_s` factor rescales Sampson (pixel-distance units) into a range comparable to reprojection (pixel²).

4. **Local optimization** (LO): once a candidate becomes the new best, a C++ Ceres-auto-differentiated least-squares minimizes the joint objective over **all** inliers:

$$E(\Theta) = \sum_{I_1} E_{r(1→2)} + \sum_{I_2} E_{r(2→1)} + 2λ_s \frac{τ_r}{τ_s} \sum_{I_3} E_s$$

with shared parameters `Θ = (R, t, α, β_1, β_2, f, …)`. LO tightens the best hypothesis using much more than the minimal sample.

5. **Termination**: standard RANSAC confidence threshold on inlier count.

Graceful-degradation guarantee: when all depth priors are wrong, `I_1` and `I_2` shrink, solver-selection probability shifts to point-based, and the estimator converges to standard Sampson-scored RANSAC — no hard failure mode.

## Why it wins
Causal story: a pure depth-aware estimator (depth-only solver + depth-only score) collapses when depth priors are unreliable (outdoor MegaDepth, foliage, reflections). A pure point-based estimator ignores the ~40–50% of correspondences where depth priors are in fact accurate and could provide metric constraints. The hybrid estimator's selection-on-inlier-ratios scheme lets each regime dominate where appropriate, and the hybrid LO solves jointly so the final pose + corrections are coherent across modalities.

Isolating ablations (Table 7 of [[yu2025_madpose]], ScanNet-1500 shared-focal, SP+SG, DA-met, AUC@10°):
- H solver + H LO + H score (full): 37.54.
- D/H/H (depth-aware solver only): 36.17 (−1.4).
- P/H/H (point solver only): 35.35 (−2.2).
- H/D/H (depth-only LO): 34.39 (−3.2).
- P/P/P (classical RANSAC): 30.27 (−7.3).

Full hybrid beats every ablation cleanly.

Full-system gains across matchers/MDE (Tables 1–5 of [[yu2025_madpose]]):
- ScanNet-1500 calibrated, SP+LG, MoGe: 42.18 vs 39.11 PoseLib-5pt (+3 AUC@10°).
- ScanNet-1500 shared-focal, MASt3R, DA-met: 56.99 vs 30.27 PoseLib-6pt (+27).
- MegaDepth-1500 two-focal, MASt3R, DA-met: 53.11 vs 36.80 PoseLib-7pt (+16).

Runtime: 31 ms calibrated / 65 ms shared-focal / 129 ms two-focal on CPU with SP+LG matches.

## Preconditions & compatibility
- **Upstream**: requires both a depth-aware solver ([[affine-corrected-minimal-relative-pose-solvers_yu2025]]) and a classical point-based solver (Nistér 5pt / Stewénius 6pt / 7pt fundamental). Both must be callable within a RANSAC loop. Paper uses PoseLib for the point-based solvers.
- **Downstream**: outputs `(R, t)` up to scale, plus `(α, β_1, β_2, f_1, f_2)` and inlier mask. Focal lengths are useful for downstream SfM initialization; affine corrections are currently unused downstream but could be propagated (see open questions).
- **Implementation**: C++ with RansacLib, Ceres for LO, pybind11 Python bindings.

## Trade-offs vs. the decomposed pipeline
Not applicable — `stage-swap` scope on a single hybrid-robust-estimator stage. The alternative (two separate RANSAC passes, one depth-aware one point-based, picking the winner) is conceptually simpler but empirically weaker because the **joint LO over all inlier types** is what closes the final accuracy gap (Table 7 shows the LO ablation alone costs 3.2 AUC@10°).

## Pipeline-shape implications
Bundle node: adopting threads must honor the `co_requires:` bundle — the hybrid estimator is useless without its depth-aware solver and depth-induced reprojection scoring. When written into a thread's SOTA pipeline, this node appears bundled with its two co-required ideas, all filling a single composite node at [[pose-estimation.hybrid-robust-estimator]].

## Open questions
- **SfM integration**: can the hybrid estimator replace COLMAP's 5pt two-view init inside incremental SfM, propagating `α, β` estimates across pairs that share a view? The per-pair nature of the estimator is currently the integration blocker. Open.
- **Feed-forward depth input**: when the MDE model is DUSt3R / MASt3R (which predict depth implicitly), should the predicted *pointmap* (not just depth) feed the solver directly? Untested.
- **Degenerate-scene robustness theory**: the graceful-degradation story is empirical. Are there adversarial configurations (e.g., depth prior confident but wrong while epipolar is ambiguous) where the selection-on-ratios rule is biased? Not analyzed.
- **Matcher ablation for outliers**: with very weak matchers (SIFT + NN, no SuperPoint), does the hybrid estimator still win, or does it inherit the matcher's failures? Table 1 tests only strong matchers.
