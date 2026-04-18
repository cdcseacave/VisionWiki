---
title: LLM-structured-scenes LLM decoding
type: stage
slug: llm-structured-scenes.llm-decoding
consumes: [voxel-tokens, fine-tuned-llm]
produces: [python-like-scene-script]
invariants: [structured-output-parseable]
provides_properties: [human-readable-executable-looking-output]
requires_upstream_properties: [llm-fine-tuned-on-spatial-tasks]
data_regime: [indoor-scenes]
tags: [qwen25, structured-output, spatiallm]
created: 2026-04-18
updated: 2026-04-18
---

Fine-tuned LLM emits a Python-like scene script (walls, doors, windows, bboxes). Example fillers: [[sonata-llm-structured-scene-scripts_mao2025]].
