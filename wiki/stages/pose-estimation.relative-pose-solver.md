---
title: Relative-pose minimal solver
type: stage
slug: pose-estimation.relative-pose-solver
consumes: [pixel-correspondences, optional-per-view-depth-priors, optional-intrinsics]
produces: [relative-pose-hypotheses, optional-affine-depth-corrections, optional-focal-lengths]
invariants: [hypotheses-consistent-with-minimal-correspondence-count, up-to-global-scale]
provides_properties: [minimal-sample-pose-hypotheses-for-robust-estimator]
requires_upstream_properties: [matched-correspondences-above-minimal-count]
data_regime: [two-view, static-scene]
tags: [pose-estimation, minimal-solver, ransac-compatible, depth-prior-optional]
created: 2026-04-21
updated: 2026-04-21
---

Minimal (or minimum-sampling) solver that turns a small set of pixel correspondences (optionally with per-view depth priors and/or intrinsics) into one or more relative-pose hypotheses `(R, t)`. Sits inside a robust estimator ([[pose-estimation.hybrid-robust-estimator]] or classical RANSAC) which samples, hypothesizes, and scores.

Axis of variation:

- **Calibration**: `calibrated` (intrinsics known), `shared-focal` (unknown `f` common to both views), `two-focal` (`f₁, f₂` both unknown).
- **Depth usage**: `point-only` (classical 5pt / 6pt / 7pt), `depth-aware-scale-only` (e.g. 2pt+D, 3p3d), `depth-aware-affine` (scale + per-view shift).

Example fillers:

- *5-point Nistér 2003* — classical calibrated essential-matrix, no depth.
- *6-point Stewénius 2008* — shared-focal fundamental matrix.
- *7-point fundamental* — two-focal fundamental matrix.
- *2pt+D (Barath 2022)* — calibrated depth-aware, scale-only; rank-deficient for rigid alignment.
- *3p3d / 4p4d (Ding 2024)* — fundamental-matrix-with-depths, scale-only.
- [[affine-corrected-minimal-relative-pose-solvers_yu2025]] — calibrated 3pt, shared-focal 4pt, two-focal 4pt; solves scale + per-view shifts jointly, up to 4–8 solutions.

Valid-filler constraints: must return a bounded number of pose hypotheses per sample (otherwise scoring stage explodes); must be degeneracy-aware (the robust wrapper can tolerate some ambiguity but not silent divergence on coplanar samples).
