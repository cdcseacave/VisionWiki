---
title: Feature-matching uncertainty (per-match precision)
type: stage
slug: feature-matching.uncertainty
consumes: [per-match-features]
produces: [per-match-2x2-precision-matrix]
invariants: [anisotropic-uncertainty-captured]
provides_properties: [weighted-sampson-ready]
requires_upstream_properties: [dense-matcher-output]
data_regime: [matcher-inference]
tags: [roma-v2, cholesky-precision, covariance]
created: 2026-04-18
updated: 2026-04-18
---

Per-pixel Cholesky-parameterized 2×2 precision matrix for anisotropic match uncertainty. The input the "multi-prior Jacobian fusion" bet (Bet #010) needs. Example fillers: [[roma-v2-predictive-covariance_edstedt2025]].
