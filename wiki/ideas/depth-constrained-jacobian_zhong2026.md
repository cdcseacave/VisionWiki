---
title: Depth-constrained Jacobian structure in GP + BA
type: idea
source_paper: wiki/papers/zhong2026_instantsfm.md
also_in: []

scope: stage-swap
stages: [sfm.global-positioning, sfm.bundle-adjustment]
collapses: []
splits_into: []
rewrites: {}

inputs: [feature-matches, per-observation-metric-depth, camera-intrinsics]
outputs: [depth-augmented-jacobian, refined-camera-poses, metric-scale-points]
assumptions: [posed-or-initialized-cameras, metric-depth-available-per-observation, static-scene]
requires_upstream_property: [metric-depth-per-observation, reprojection-residuals-available]
requires_downstream_property: [lm-solver-accepts-heterogeneous-residuals]
learned_params: []
failure_modes: [bad-depth-prior-biases-pose, depth-sparsity-leaves-rank-deficient-regions]

requires: []
unlocks: []
co_requires: [dynamic-parameter-extraction_zhong2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, bundle-adjustment, depth-prior, gpu-native]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

In global positioning (GP), per-observation scale variables that would normally be free-parameters are pinned to the known metric depth `d_i`, propagating metric scale through shared camera-center columns in the Jacobian. In BA, an additional depth residual `r_d = d_rendered - d_prior` is appended to the reprojection residual stack, weighted by a binary validity mask so invalid / missing depth observations contribute zero. The full Jacobian is a stacked [reprojection; depth] structure that LM can solve uniformly on GPU.

## Why it wins

Metric scale is preserved end-to-end — no post-hoc scale alignment needed. Depth priors act as first-class residuals rather than regularization, so their gradient signal is on equal footing with reprojection. Paper's DTU result (PSNR 24.57 vs. GLOMAP's 17.98) isolates the contribution: depth-constrained Jacobian lets InstantSfM solve scenes where GLOMAP fails outright.

## Preconditions & compatibility

Requires a metric-depth source — [[monocular-depth-estimation]] (Metric3Dv2, DepthAnything) or RGB-D sensor. Bundles with dynamic parameter extraction (`co_requires:`) because without it, depth-augmented Jacobians still hit rank deficiency when outliers temporarily invalidate points.

## Open questions

- Sensitivity to depth-prior quality (bad monocular depth may bias pose).
- Multi-prior fusion (depth + normal + match confidence) — Bet #010 is the natural extension.
