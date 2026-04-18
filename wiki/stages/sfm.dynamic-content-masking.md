---
title: SfM dynamic-content masking
type: stage
slug: sfm.dynamic-content-masking
consumes: [video-or-image-stream, optional-flow-or-segmentation]
produces: [per-pixel-motion-probability]
invariants: [static-content-passes-through-untouched]
provides_properties: [ba-operates-only-on-static]
requires_upstream_properties: [learned-motion-predictor-available]
data_regime: [dynamic-scenes, casual-video]
tags: [sfm, dynamic-scene, motion-segmentation, megasam]
created: 2026-04-18
updated: 2026-04-18
---

Masks moving objects out of BA's supervision. Learned per-pixel motion-probability (MegaSaM) is the canonical filler. Example fillers: [[learned-motion-probability-dynamic-ba_li2025]].
