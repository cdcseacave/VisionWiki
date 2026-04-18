---
title: MVS feature aggregation
type: stage
slug: mvs.feature-aggregation
consumes: [n-warped-feature-volumes]
produces: [unified-cost-volume]
invariants: [permutation-invariance-in-n]
provides_properties: [arbitrary-n-handled]
requires_upstream_properties: [features-per-view, known-poses]
data_regime: [learned-mvs]
tags: [mvs, mvsnet, cost-volume, variance-aggregation]
created: 2026-04-18
updated: 2026-04-18
---

Aggregates N warped feature volumes into one (variance aggregation — MVSNet). Attention-based alternatives follow in later papers; variance remains the permutation-invariant baseline. Example fillers: [[plane-sweep-variance-cost-volume_yao2018]], [[stereo-to-multiview-inference-time-warping_chebbi2025]].
