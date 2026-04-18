---
title: Radiance-fields incremental capture
type: stage
slug: radiance-fields.incremental-capture
consumes: [rgbd-stream-or-image-stream, real-time-budget]
produces: [growing-primitive-set-online, pose-graph]
invariants: [o-n-memory-per-submap]
provides_properties: [submap-bounded-memory, loop-closure-compatible]
requires_upstream_properties: [depth-or-monocular-depth-completion]
data_regime: [online, sequential-capture]
tags: [3dgs-slam, submaps, online]
created: 2026-04-18
updated: 2026-04-18
---

Online reconstruction: as frames arrive, allocate new primitives / submaps. VPGS-SLAM is the canonical filler. Must cooperate with [[radiance-fields.global-consistency]] for loop closure. Example fillers: [[voxel-anchor-progressive-submaps_deng2026]].
