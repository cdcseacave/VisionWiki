---
title: LLM-structured-scenes point-cloud tokenization
type: stage
slug: llm-structured-scenes.point-cloud-tokenization
consumes: [point-cloud-from-rgbd-or-slam]
produces: [voxel-token-stream-for-llm]
invariants: [voxel-resolution-fits-llm-context-budget]
provides_properties: [llm-readable-3d-input]
requires_upstream_properties: [point-cloud-source]
data_regime: [indoor-point-cloud]
tags: [sonata, point-cloud-encoder, spatiallm]
created: 2026-04-18
updated: 2026-04-18
---

Tokenizes a point cloud (2.5cm voxel tokens in SpatialLM) for LLM input. Example fillers: [[sonata-llm-structured-scene-scripts_mao2025]].
