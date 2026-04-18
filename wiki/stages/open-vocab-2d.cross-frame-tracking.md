---
title: Open-vocab-2d cross-frame tracking
type: stage
slug: open-vocab-2d.cross-frame-tracking
consumes: [per-frame-masks, per-frame-features]
produces: [per-instance-id-across-frames]
invariants: [stable-ids-under-moderate-occlusion]
provides_properties: [video-segmentation-native]
requires_upstream_properties: [mask-source-per-frame]
data_regime: [video]
tags: [sam3, video-tracking, instance-ids]
created: 2026-04-18
updated: 2026-04-18
---

Propagates instance IDs across video frames. SAM 3's memory-based tracker is the canonical filler. Example fillers: [[sam3-native-video-ids_carion2026]].
