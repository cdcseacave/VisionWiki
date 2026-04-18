---
title: Dynamic parameter extraction for rank-stable LM iterations
type: idea
source_paper: wiki/papers/zhong2026_instantsfm.md
also_in: []

scope: drop-in
stages: [sfm.lm-solver]
collapses: []
splits_into: []
rewrites: {}

inputs: [current-jacobian, per-observation-validity-mask]
outputs: [compact-reparameterized-jacobian, index-remap]
assumptions: [outlier-rate-bounded-per-iteration, reparameterization-cheap]
requires_upstream_property: [per-observation-validity-computable]
requires_downstream_property: [consumer-can-remap-indices]
learned_params: []
failure_modes: [index-remap-overhead-dominates-on-tiny-problems]

requires: []
unlocks: [depth-constrained-jacobian_zhong2026]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, lm-solver, rank-stability, gpu-native]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

At each Levenberg-Marquardt iteration: identify the currently-valid observations (those passing outlier checks), extract the *compact* set of cameras and 3D points that still appear in ≥1 valid observation, and remap their indices into a dense parameter vector. Solve LM on the compact problem; expand the update back to the full parameter space after the step. The Jacobian on the compact problem has no all-zero columns → no rank deficiency → no solver failure.

## Why it wins

Classical LM solvers use damping hacks (increase λ) when normal equations become rank-deficient due to outliers. Dynamic parameter extraction fixes the root cause: the parameters causing rank deficiency are simply not in the solve. Paper reports robust convergence where COLMAP/GLOMAP outright fail (most ScanNet scenes). Cheap on GPU because the reparameterization is a scatter-gather, not a solve.

## Preconditions & compatibility

Requires a GPU-native sparse-op framework (PyTorch sparse ops or equivalent). The index-remap cost is ~constant in compact-problem size; relative overhead dominates only for tiny problems (<100 cameras).

## Open questions

- Does the same extraction pattern work for factor-graph solvers (Ceres, GTSAM)?
- Interaction with robust-loss warmup (CuSfM-style two-stage) untested.
