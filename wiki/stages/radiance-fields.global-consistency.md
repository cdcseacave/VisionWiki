---
title: Radiance-fields global consistency
type: stage
slug: radiance-fields.global-consistency
consumes: [submap-set, pose-graph, loop-candidates]
produces: [globally-consistent-primitive-union, closed-loop-poses]
invariants: [local-submap-consistency-preserved]
provides_properties: [loop-closure, drift-bounded]
requires_upstream_properties: [pose-graph-with-loop-detection]
data_regime: [online-or-offline-multi-submap]
tags: [3dgs-slam, loop-closure, submap-fusion]
created: 2026-04-18
updated: 2026-04-18
---

Corrects drift across submaps via loop closure + submap fusion. Fillers must preserve lifted features across the fusion (see [[lifting-foundation-models.online-voxel-feature-fusion]]). Example fillers: [[voxel-anchor-progressive-submaps_deng2026]].
