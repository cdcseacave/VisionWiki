---
title: VLM-reasoning view retrieval
type: stage
slug: vlm-reasoning.view-retrieval
consumes: [pre-trained-3dgs-scene, query]
produces: [top-k-candidate-views-with-information-gain-scores]
invariants: [retrieval-informed-by-query]
provides_properties: [scene-as-queryable-database]
requires_upstream_properties: [3dgs-trained]
data_regime: [inference-time]
tags: [gaussexplorer, view-retrieval]
created: 2026-04-18
updated: 2026-04-18
---

Picks the top-K views most likely to inform the query. Example fillers: [[vlm-view-retrieval-reasoning-3dgs_kim2026]].
