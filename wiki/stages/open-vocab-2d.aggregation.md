---
title: Open-vocab-2d aggregation
type: stage
slug: open-vocab-2d.aggregation
consumes: [clip-semantic-scores, dino-affinity, sam-masks]
produces: [final-open-vocab-segmentation]
invariants: [three-signals-combined-losslessly]
provides_properties: [modular-component-swaps]
requires_upstream_properties: [all-three-inputs-from-upstream-stages]
data_regime: [trident-style]
tags: [trident, splice-then-segment, clip-dino-sam]
created: 2026-04-18
updated: 2026-04-18
---

Top-level assembly of CLIP + DINO + SAM. Trident's "Splice-then-Segment" is the canonical filler; replaces earlier Segment-then-Splice. Example fillers: [[trident-splice-then-segment_shi2024]].
