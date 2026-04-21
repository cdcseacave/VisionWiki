---
title: Hybrid robust pose estimator
type: stage
slug: pose-estimation.hybrid-robust-estimator
consumes: [correspondences, optional-depth-priors, solver-pool, scoring-pool]
produces: [final-relative-pose, affine-corrections, inlier-set]
invariants: [graceful-degradation-when-any-one-prior-fails, bounded-ransac-iterations]
provides_properties: [pose-quality-bounded-by-best-available-modality]
requires_upstream_properties: [at-least-one-relative-pose-solver, at-least-one-scoring-function]
data_regime: [two-view, static-scene, depth-priors-possibly-unreliable]
tags: [pose-estimation, lo-ransac, lo-msac, hybrid, multi-modal]
created: 2026-04-21
updated: 2026-04-21
---

Robust-estimator wrapper that combines **multiple minimal solvers** and **multiple scoring functions** in a single LO-RANSAC / LO-MSAC loop. Unlike classical RANSAC (fixed solver + Sampson scoring), the hybrid estimator probabilistically selects which solver to sample from and which score to apply per correspondence, so that when one modality (e.g., depth prior) fails the other (point-only epipolar) still drives convergence.

Components a filler wires together:

- **Solver pool**: ≥2 instances of [[pose-estimation.relative-pose-solver]] with different modalities (point-only + depth-aware is the common pairing).
- **Scoring pool**: ≥2 instances of [[pose-estimation.robust-estimator-scoring]] (e.g., Sampson + depth-induced reprojection).
- **Selection policy**: per-iteration probability for which solver to sample, updated from inlier-type ratios across iterations.
- **Local optimization**: joint least-squares over all shared parameters (pose + affine corrections + optional focals) minimizing a weighted sum of the active scores.

Example fillers:

- *Hybrid LO-RANSAC (Camposeco 2018)* — the original multi-solver framework; used for pose + focal length.
- [[hybrid-lo-msac-dual-modality-estimator_yu2025]] — MADPose's instantiation: depth-aware solver + point-based solver + depth-induced-reprojection score + Sampson score, three inlier types.

Valid-filler constraints: filler must handle depth-prior absence (no-op to point-only fallback); must report which data type each final inlier belonged to (needed for downstream confidence weighting).
