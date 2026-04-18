---
title: SfM next-view scheduling
type: stage
slug: sfm.next-view-scheduling
consumes: [current-reconstruction, remaining-view-candidates]
produces: [next-view-to-register]
invariants: [wasted-ba-cycles-minimized]
provides_properties: [expected-gain-based-ordering]
requires_upstream_properties: [current-reconstruction-state]
data_regime: [incremental-sfm]
tags: [sfm, scheduling, incremental, colmap]
created: 2026-04-18
updated: 2026-04-18
---

Picks which view to register next. Expected-gain scoring (COLMAP) beats naïve match-count heuristics. Example fillers: [[colmap-incremental-sfm_schonberger2016]].
