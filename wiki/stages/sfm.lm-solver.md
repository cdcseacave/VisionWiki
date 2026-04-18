---
title: SfM LM / GN solver
type: stage
slug: sfm.lm-solver
consumes: [jacobian-residual-pair]
produces: [parameter-update]
invariants: [full-rank-normal-equations]
provides_properties: [outlier-rate-resilience]
requires_upstream_properties: [bundle-adjustment-formulation-defined]
data_regime: [inner-loop-of-ba]
tags: [lm, gauss-newton, rank-stability]
created: 2026-04-18
updated: 2026-04-18
---

Inner-loop Levenberg-Marquardt / Gauss-Newton step. Dynamic parameter extraction (compact re-indexing per iteration) prevents rank-deficient solves when outliers temporarily invalidate parameters. Example fillers: [[dynamic-parameter-extraction_zhong2026]].
