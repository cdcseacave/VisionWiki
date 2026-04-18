---
title: Lifting — primitive refinement (boundary-aware)
type: stage
slug: lifting-foundation-models.primitive-refinement
consumes: [primitives-with-identity, projected-mask-boundaries]
produces: [boundary-clean-primitives]
invariants: [cross-boundary-primitives-cut-or-split]
provides_properties: [clean-per-instance-identity-despite-noisy-masks]
requires_upstream_properties: [primitives-with-identity]
data_regime: [in-the-wild, inconsistent-masks]
tags: [seg-wild, spiky-cutter, sgc]
created: 2026-04-18
updated: 2026-04-18
---

Refines primitives that straddle 2D mask boundaries (Spiky 3D Gaussian Cutter). Example fillers: [[spiky-3d-gaussian-cutter_bao2025]].
