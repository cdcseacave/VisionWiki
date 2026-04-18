---
title: SfM bundle adjustment
type: stage
slug: sfm.bundle-adjustment
consumes: [initial-cameras, 3d-points, feature-matches, optional-depth-or-normal-priors]
produces: [refined-cameras, refined-points]
invariants: [reprojection-residual-minimized, jacobian-sparse-structure-preserved]
provides_properties: [metric-accuracy, rank-stable-solve]
requires_upstream_properties: [sfm-initialization]
data_regime: [static-scene, posed-or-to-be-posed]
tags: [sfm, ba, ceres, lm, depth-priors]
created: 2026-04-18
updated: 2026-04-18
---

Refines cameras + 3D points via LM / Gauss-Newton. Priors enter as additional residual blocks (depth-constrained Jacobian, normal integration). Bundle with dynamic parameter extraction (`co_requires:`) to prevent rank deficiency. Example fillers: [[depth-constrained-jacobian_zhong2026]], [[learned-motion-probability-dynamic-ba_li2025]].
