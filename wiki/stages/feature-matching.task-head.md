---
title: Feature-matching task head (dense matcher)
type: stage
slug: feature-matching.task-head
consumes: [image-pair, frozen-backbone-features]
produces: [dense-correspondence-map]
invariants: [subpixel-accuracy]
provides_properties: [inference-speed-via-coarse-stride]
requires_upstream_properties: [dinov3-or-equivalent-backbone]
data_regime: [matching-pair-inference]
tags: [roma-v2, dense-matcher, dinov3]
created: 2026-04-18
updated: 2026-04-18
---

Dense feature matcher on top of a frozen backbone. RoMa v2 (alternating frame-wise / global attention + custom CUDA) is the canonical filler. Example fillers: [[roma-v2-dinov3-dense-matcher_edstedt2025]].
