---
title: Robust-estimator scoring function
type: stage
slug: pose-estimation.robust-estimator-scoring
consumes: [pose-hypothesis, correspondences, optional-depth-priors-with-corrections]
produces: [per-correspondence-score]
invariants: [score-bounded, monotone-in-residual]
provides_properties: [inlier-partition-for-loransac-lomsac]
requires_upstream_properties: [pose-hypothesis-from-minimal-solver]
data_regime: [inside-ransac-lomsac-loop]
tags: [pose-estimation, ransac, msac, sampson, depth-induced-reprojection]
created: 2026-04-21
updated: 2026-04-21
---

Assigns a score to each correspondence given a pose hypothesis, used by the outer robust estimator to rank hypotheses and partition inliers. Classical form is the Sampson epipolar error; depth-aware variants reproject lifted 3D points.

Example fillers:

- *Sampson error* — first-order epipolar distance approximation; pose-only, depth-agnostic. The RANSAC-era default for relative pose.
- *Symmetric reprojection* — lift 3D points under known depth + pose, project across views, take min-of-both-directions. Needs depth.
- [[depth-induced-reprojection-scoring_yu2025]] — symmetric reprojection under *solved* affine-corrected depth (α, β₁, β₂ from the minimal solver). Truncated MSAC form with per-direction thresholds.
- *Combined Sampson + reprojection* (MADPose hybrid): uses whichever score is tighter per-correspondence; correspondences with bad depth priors still contribute via the Sampson channel.

Valid-filler constraints: score must be computable in constant time per correspondence (RANSAC iterates thousands of times); score must degrade gracefully when depth priors are unreliable (otherwise the filler is fragile on outdoor / texture-poor data).
