---
title: MVS depth refinement
type: stage
slug: mvs.depth-refinement
consumes: [coarse-depth-map, reference-image]
produces: [edge-aligned-residual-corrected-depth]
invariants: [reference-image-guides-edges]
provides_properties: [high-frequency-detail-preservation]
requires_upstream_properties: [initial-depth-available]
data_regime: [learned-mvs]
tags: [mvs, color-guided-refinement, mvsnet]
created: 2026-04-18
updated: 2026-04-18
---

Color-guided refinement of coarse depth. Example fillers: [[plane-sweep-variance-cost-volume_yao2018]].
