---
title: SfM view-graph construction
type: stage
slug: sfm.view-graph-construction
consumes: [image-set, optional-prior-trajectory]
produces: [view-graph-edges]
invariants: [coverage-sufficient-for-connectivity]
provides_properties: [minimal-redundant-edges]
requires_upstream_properties: [feature-extraction-done]
data_regime: [unordered-or-sequential-images]
tags: [sfm, view-graph, slam-prior, cusfm]
created: 2026-04-18
updated: 2026-04-18
---

Builds the graph of which image pairs to match. All-pairs (COLMAP default) vs. minimal-graph-from-prior-trajectory (CuSfM). Example fillers: [[cusfm-slam-prior-factor-graph_yu2025]].
