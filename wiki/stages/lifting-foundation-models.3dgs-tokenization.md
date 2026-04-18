---
title: Lifting — 3DGS tokenization
type: stage
slug: lifting-foundation-models.3dgs-tokenization
consumes: [3dgs-scene]
produces: [token-stream-for-transformer]
invariants: [all-per-primitive-attrs-captured]
provides_properties: [transformer-friendly-input]
requires_upstream_properties: [3dgs-available]
data_regime: [pre-computed-3dgs]
tags: [gs-tokenizer, clip-gs]
created: 2026-04-18
updated: 2026-04-18
---

Serializes a 3DGS scene into a token stream (FPS centers + k-NN patches). Example fillers: [[gs-tokenizer-scene-clip-contrastive_jiao2025]].
