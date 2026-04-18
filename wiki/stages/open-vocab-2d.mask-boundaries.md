---
title: Open-vocab-2d mask boundaries
type: stage
slug: open-vocab-2d.mask-boundaries
consumes: [image, coarse-score-or-prompt]
produces: [high-quality-mask-boundaries]
invariants: [spatially-coherent-masks]
provides_properties: [clean-boundaries-snapped-to-signal]
requires_upstream_properties: [segmentation-foundation-model]
data_regime: [any-image]
tags: [sam, sam3, promptable-segmentation]
created: 2026-04-18
updated: 2026-04-18
---

Snaps aggregated semantic scores to clean mask boundaries. SAM's promptable ambiguity-aware head is the canonical filler; SAM 3 supersedes for concept prompts. Example fillers: [[sam-promptable-multi-mask_kirillov2023]], [[sam3-native-video-ids_carion2026]].
