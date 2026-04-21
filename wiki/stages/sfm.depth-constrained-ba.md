---
title: Depth-constrained bundle adjustment
type: stage
slug: sfm.depth-constrained-ba
consumes: [poses, scene-points, per-view-monocular-depth, per-view-monocular-normals, per-view-uncertainties]
produces: [refined-poses, refined-scene-points, refined-per-view-depth-maps]
invariants: [reprojection-consistency, depth-point-consistency, normal-integration-consistency]
provides_properties: [robust-to-noisy-depth-priors, propagates-prior-uncertainty]
requires_upstream_properties: [calibrated-or-recalibrated-per-pixel-depth-uncertainty]
data_regime: [incremental-sfm, global-sfm, pose-and-depth-joint-refinement]
tags: [sfm, bundle-adjustment, mono-depth, normal-integration]
created: 2026-04-21
updated: 2026-04-21
---

A bundle adjustment stage whose objective is not reprojection alone but the sum of three terms: reprojection $C_{BA}$, depth regularization tying 3D points to refined per-view depth maps $C_{reg}$, and depth integration tying refined depths to the prior depth + surface-normal gradient consistency $C_{int}$. Robustified with truncated-L2 / Cauchy losses; uncertainty-weighted via propagated covariances from monocular prior.

The joint Hessian is not amenable to the Schur complement trick (normal integration couples off-diagonals of the 3D-point block). Typical solvers use alternating block coordinate descent: minimize $C_{reg} + C_{int}$ per image (GPU), then $C_{BA} + C_{reg}$ across views (CPU Ceres).

Distinct from [[sfm.bundle-adjustment]] (reprojection-only). Fills: [[bilateral-normal-integration-with-uncertainty_pataki2025]] as the $C_{int}$ sub-term; overall filled by [[mono-depth-normal-constrained-incremental-sfm_pataki2025]].

Example fillers:
- [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] — depth + normal both.
- [[depth-constrained-jacobian_zhong2026]] — InstantSfM variant, depth-only, no normal integration, GPU-native.
