---
title: Lifting — 3D identity regularization
type: stage
slug: lifting-foundation-models.3d-identity-regularization
consumes: [primitives-with-identity, depth-normal-fields]
produces: [spatial-consistency-loss]
invariants: [depth-normal-boundaries-preserved]
provides_properties: [clean-per-instance-identity-fields]
requires_upstream_properties: [per-primitive-identity-available]
data_regime: [per-scene]
tags: [3d-regularization, spatial-consistency, gaussian-grouping]
created: 2026-04-18
updated: 2026-04-18
---

Regularizes identity fields to respect 3D structure (depth/normal boundaries = instance boundaries). Gaussian Grouping's canonical filler. Example fillers: [[gaussian-grouping-spatial-consistency_ye2024]].
