---
title: Lifting — cross-view ID consistency
type: stage
slug: lifting-foundation-models.cross-view-id-consistency
consumes: [per-view-sam-masks, per-view-features]
produces: [per-mask-track-id]
invariants: [same-instance-gets-same-id-across-views]
provides_properties: [supervision-consistency-for-per-primitive-identity]
requires_upstream_properties: [per-view-masks-and-features]
data_regime: [multi-view-capture]
tags: [cross-view-tracking, iou-similarity, sam3-native]
created: 2026-04-18
updated: 2026-04-18
---

Associates 2D masks across views. Heuristic IoU+feature-similarity (Gaussian Grouping) vs. native SAM 3 video IDs. Example fillers: [[cross-view-mask-association-iou-similarity_ye2024]], [[sam3-native-video-ids_carion2026]].
