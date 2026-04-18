---
title: SfM iterative triangulation + BA loop
type: stage
slug: sfm.iterative-triangulation-ba
consumes: [matches, partial-reconstruction]
produces: [refined-reconstruction-with-recovered-tracks]
invariants: [track-completeness-improves-per-iteration]
provides_properties: [tracks-recovered-after-pose-refinement]
requires_upstream_properties: [ba-and-triangulation-available]
data_regime: [final-polish-of-incremental-or-global-sfm]
tags: [sfm, ba, triangulation, colmap]
created: 2026-04-18
updated: 2026-04-18
---

Alternates triangulation and BA until convergence, recovering tracks lost to initial noise. The accuracy-reference inner loop for classical SfM — InstantSfM/CuSfM keep this loop. Example fillers: [[colmap-incremental-sfm_schonberger2016]].
