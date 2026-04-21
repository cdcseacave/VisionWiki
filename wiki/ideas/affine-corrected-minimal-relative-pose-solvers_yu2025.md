---
title: Affine-corrected minimal relative-pose solvers (calibrated 3pt / shared-focal 4pt / two-focal 4pt)
type: idea
source_paper: wiki/papers/yu2025_madpose.md
also_in: []

scope: stage-swap
stages: [pose-estimation.relative-pose-solver]
collapses: []
splits_into: []
rewrites: {}

inputs: [minimal-pixel-correspondences, per-view-mono-depth-samples, optional-intrinsics]
outputs: [relative-pose-hypotheses, per-view-affine-corrections-alpha-beta1-beta2, optional-focal-lengths]
assumptions: [affine-invariant-or-metric-mono-depth-available, static-pair, per-view-shift-non-zero-in-practice]
requires_upstream_property: [mono-depth-consistent-within-view-up-to-affine-transform]
requires_downstream_property: [robust-estimator-handles-up-to-4-or-8-solutions]
learned_params: []
failure_modes: [near-coplanar-3-point-samples-degenerate, α-approaching-zero-numerical-instability, β-larger-than-scene-extent-nonphysical-solutions]

requires: []
unlocks: [depth-induced-reprojection-scoring_yu2025, hybrid-lo-msac-dual-modality-estimator_yu2025]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [pose-estimation, minimal-solver, mono-depth, affine-correction, grobner-basis, calibrated, shared-focal, two-focal]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
Given pixel correspondences `(p_{1j}, p_{2j})` with per-view mono-depth samples `d_{ij} = D_i(p_{ij})`, model each view's depth as `D̂_i = a_i D_i + b_i` (affine-invariant). The relative-pose has global scale freedom, so set `α = a₂/a₁` and `β_i = b_i / a_i`, giving `d̂_{1j} = d_{1j} + β_1`, `d̂_{2j} = α(d_{2j} + β_2)`. Lifted 3D points are `P_{ij} = K_i^{-1} [p_{ij}^\top, 1]^\top \cdot d̂_{ij}`.

Since `P_{2j} = R P_{1j} + t` is a rigid transform, pairwise distances are preserved:

$$\|P_{1j} - P_{1k}\|^2 = \|P_{2j} - P_{2k}\|^2$$

Expand both sides. The left side is a quadratic in `(β_1)`, the right in `(α, β_2)`. Taking pairs over `M` correspondences gives `C(M, 2)` equations **in only `(α, β_1, β_2)` (and focals when unknown)** — rotation and translation drop out. This is the key observation: the length-preservation constraint lets you decouple affine-correction solving from pose solving.

**Calibrated 3-point**: `M=3` → `C(3,2) = 3` equations in `(α, β_1, β_2)`. Reparametrize `γ = α²` to reduce the quartics to cubic order. Apply the Larsson 2017 Gröbner-basis template ([34] in refs): 12×12 linear elimination + 4×4 eigenvalue → **up to 4 real solutions**.

**Shared-focal 4-point**: `M=4` with shared unknown focal `f`. Reparametrize `ω = 1/f²`. Pick 4 of the 6 length-preservation equations. 36×36 template → **up to 8 solutions**.

**Two-focal 4-point**: `M=4` with independent focals `f_1, f_2`. `ω_i = 1/f_i²`. Pick 5 of 6 equations, set up 40×40 linear elimination + 4×4 eigenvalue → **up to 4 solutions**.

Given each real, positive-depth `(α, β_1, β_2, …)` solution, recover `(R, t)` via Procrustes / SVD on lifted 3D points (Schönemann 1966): the back-projection is fully determined, so alignment is closed-form. Final output per minimal sample: up to 4 or 8 hypotheses `(R, t, α, β_1, β_2, [f, f_1, f_2])`.

No learned parameters — the solvers are purely algebraic derivations of the length-preservation constraint under the affine-invariant depth model. Larsson 2017's automated Gröbner-basis generator handles the polynomial manipulation.

## Why it wins
Causal story: prior depth-aware solvers (Barath 2022 2pt+D; Ding 2024 3p3d / 4p4d) model the depth prior as **scale-invariant only** (`β = 0`). Figure 6 of [[yu2025_madpose]] shows that fitted `β` to ground-truth depth is routinely `> 10%` of median depth even for *metric*-trained DA — so the scale-only assumption is empirically wrong, and the resulting rigid alignment (3-point Procrustes) is rank-deficient. Explicitly solving for `β_1, β_2` fixes the rank deficiency and makes the 3-point solver well-posed.

Isolating ablations (Table 6 of [[yu2025_madpose]], ScanNet-1500 calibrated, Marigold MDE, SP+LG matches):
- Hybrid with shift: 20.68 / 40.71 / 59.92 AUC@5/10/20°.
- Hybrid, scale-only: 12.34 / 26.64 / 44.53 — **−15 AUC@20° from dropping shift**.
- Depth-only, no shift: 14.70 / 33.73 / 54.75.
- Point-only (PoseLib-5pt): 4.84 / 6.41 / 17.15.

Figure 5 shows rotation / translation error as a function of synthetic shift added to GT depth: scale-only and PnP baselines degrade sharply with added shift; the affine-correcting solvers are invariant.

Paper comparisons are strongest on outdoor and uncalibrated settings where depth priors are noisier: MegaDepth-1500 uncalibrated (Table 5), two-focal-tf solver gets 61.79 AUC@20° vs PoseLib-7pt baseline 54.89 (+7 points) and 4p4d (scale-only) 44.08 (+18 points).

## Preconditions & compatibility
- **Upstream**: any affine-invariant or metric MDE model. Tested with Depth-Anything v1/v2 (relative + metric), Marigold, Omnidata, MoGe, and MASt3R predicted depth. No specialized training required.
- **Downstream**: outputs up to 4–8 hypotheses per minimal sample, which a RANSAC / MSAC wrapper consumes as usual. Can be used with classical LO-RANSAC (PoseLib-style) or the hybrid LO-MSAC of [[hybrid-lo-msac-dual-modality-estimator_yu2025]]. Standalone usability is why `co_requires:` is empty.
- **Intrinsics**: separate solvers for calibrated / shared-focal / two-focal; choice is deployment-dependent.

## Trade-offs vs. the decomposed pipeline
Not applicable — `stage-swap` scope. Same stage input/output as classical 5pt/6pt/7pt, with per-sample compute a few ms higher due to Gröbner-basis eigenvalue step (still sub-millisecond per hypothesis).

## Open questions
- **Coplanar degeneracy**: 3-point samples on near-coplanar scene geometry produce ill-conditioned systems. Paper acknowledges but doesn't quantify the failure rate.
- **Shift propagation across a sequence**: two pairs sharing view `I_k` can disagree on `β_k`. For extension to SfM (not pair-pose), a consistency constraint across pairs is needed. Open.
- **Fusing with feed-forward depth**: DUSt3R / MASt3R predict their own depth. Paper uses their predictions as input; is there a better combination — e.g., use predicted depth distributions (not point estimates) directly in the solver's least-squares refinement?
