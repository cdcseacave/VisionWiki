---
title: LLM-structured-scenes pretraining data
type: stage
slug: llm-structured-scenes.pretraining-data
consumes: [scene-generator]
produces: [synthetic-scene-plus-annotation-pairs]
invariants: [scale-sufficient-to-pretrain-llm]
provides_properties: [closes-3d-llm-data-gap]
requires_upstream_properties: [procedural-generator]
data_regime: [training-time]
tags: [synthetic-dataset, spatiallm, pretraining]
created: 2026-04-18
updated: 2026-04-18
---

Large-scale synthetic scene corpus used to pretrain the LLM before small-real-dataset fine-tuning. Example fillers: [[large-scale-synthetic-indoor-dataset_mao2025]].
