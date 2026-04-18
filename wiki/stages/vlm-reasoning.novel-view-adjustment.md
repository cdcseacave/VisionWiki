---
title: VLM-reasoning novel-view adjustment
type: stage
slug: vlm-reasoning.novel-view-adjustment
consumes: [retrieved-views, 3dgs-scene, information-gain-criterion]
produces: [iteratively-synthesized-novel-views]
invariants: [nvs-quality-sufficient-for-vlm]
provides_properties: [coverage-of-query-relevant-viewpoints]
requires_upstream_properties: [view-retrieval-done]
data_regime: [inference-time]
tags: [gaussexplorer, nvs, information-gain]
created: 2026-04-18
updated: 2026-04-18
---

Iteratively synthesizes novel views in the neighborhood of retrieved ones. Example fillers: [[vlm-view-retrieval-reasoning-3dgs_kim2026]].
