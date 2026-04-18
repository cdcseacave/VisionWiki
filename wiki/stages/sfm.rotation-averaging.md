---
title: SfM rotation averaging
type: stage
slug: sfm.rotation-averaging
consumes: [pairwise-relative-rotations, view-graph]
produces: [global-rotations-per-camera]
invariants: [chirality-preserved]
provides_properties: [outlier-aware-with-robust-weights]
requires_upstream_properties: [pairwise-relative-poses]
data_regime: [global-sfm]
tags: [sfm, rotation-averaging, glomap]
created: 2026-04-18
updated: 2026-04-18
---

Isolates the rotation-only subproblem; robust weighting (GLOMAP) handles noisy pairwise rotations from bad view-graph edges. Example fillers: [[glomap-joint-global-positioning_pan2024]].
