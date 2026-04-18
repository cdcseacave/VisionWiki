---
title: Lifting — scene-level contrastive alignment
type: stage
slug: lifting-foundation-models.scene-level-alignment
consumes: [3dgs-token-stream, rendered-images, text]
produces: [scene-level-embedding-aligned-with-clip]
invariants: [cross-scene-transfer-preserved]
provides_properties: [zero-shot-queries-on-unseen-scenes]
requires_upstream_properties: [gs-tokenizer-output]
data_regime: [scene-level-triplets, offline-pre-training]
tags: [clip-gs, scene-contrastive, zero-shot-transfer]
created: 2026-04-18
updated: 2026-04-18
---

Scene-level contrastive alignment between 3DGS and CLIP image+text. Example fillers: [[gs-tokenizer-scene-clip-contrastive_jiao2025]].
