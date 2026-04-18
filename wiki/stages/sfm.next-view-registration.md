---
title: SfM next-view registration
type: stage
slug: sfm.next-view-registration
consumes: [current-reconstruction, candidate-new-view, matches]
produces: [registered-pose-for-new-view]
invariants: [reconstruction-connectivity-maintained]
provides_properties: [works-on-low-overlap-captures-with-mono-priors]
requires_upstream_properties: [feature-matches-to-existing-reconstruction]
data_regime: [incremental-sfm]
tags: [sfm, incremental, mp-sfm, mono-depth]
created: 2026-04-18
updated: 2026-04-18
---

Adds a new view to an in-progress reconstruction. Classical incremental SfM needs 3-view overlap; mono-depth lifting (MP-SfM) relaxes this to 2-view. Example fillers: [[mono-depth-normal-constrained-incremental-sfm_pataki2025]].
