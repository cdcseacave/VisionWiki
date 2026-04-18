---
title: Per-pixel Cholesky-parameterized 2×2 precision matrix for dense matches
type: idea
source_paper: wiki/papers/edstedt2025_roma-v2.md
also_in: []

scope: drop-in
stages: [feature-matching.uncertainty]
inputs: [matcher-features, per-pixel-matching-context]
outputs: [per-match-2x2-precision-matrix, covariance-weighted-sampson-input]
assumptions: [gaussian-error-distribution-approximately-correct, sub-pixel-accuracy-needed]
requires_upstream_property: [dense-matcher-output]
requires_downstream_property: [consumer-uses-weighted-geometry-estimation]
learned_params: [covariance-head-weights]
failure_modes: [non-gaussian-errors-misrepresented, calibration-hard-to-verify-per-scene]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [dense-matching, uncertainty, covariance, weighted-sampson]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Alongside the (u, v) match prediction, the matcher head emits a Cholesky-parameterized 2×2 precision matrix `L` such that the predicted precision is `L L^T`. The scalar covariance normally used for match weighting is upgraded to a full 2D covariance that encodes anisotropic uncertainty — e.g. a match along an edge has tight precision perpendicular to the edge, loose along it.

## Why it wins

Paper demonstrates downstream geometry estimation (Sampson error optimization) improves when the per-match covariance is used as the weight — not marginally, but enough to move metric-pose numbers. This is precisely the input the "multi-prior Jacobian fusion" bet (Bet #010) needs for its DINOv3-match-confidence residual axis.

## Preconditions & compatibility

Training assumes Gaussian error distribution — may not hold for all failure modes (textureless-sky spurious confidence is the paper's acknowledged case). Compatible with any matcher architecture that emits per-pixel output.

## Open questions

- Can the covariance head be distilled onto non-RoMa matchers (SuperGlue, LoFTR) post-hoc?
- Calibration: do the predicted precisions actually correspond to the empirical error distribution, or are they biased?
