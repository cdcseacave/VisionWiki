---
title: Generative-3d training data curation
type: stage
slug: generative-3d.training-data-curation
consumes: [synthetic-3d-pretraining-set, real-image-annotation-workflow]
produces: [large-scale-real-image-3d-training-set]
invariants: [annotator-and-model-in-the-loop]
provides_properties: [closes-3d-data-barrier]
requires_upstream_properties: [synthetic-pretraining-feasible]
data_regime: [training-time]
tags: [sam-3d, data-engine, human-in-the-loop]
created: 2026-04-18
updated: 2026-04-18
---

Data engine: model proposes, humans correct, proposals are re-ingested. Example fillers: [[real-image-data-engine_chen2025]].
