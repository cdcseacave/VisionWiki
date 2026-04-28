---
title: Feed-forward SfM attention mechanism
type: stage
slug: feed-forward-sfm.attention-mechanism
consumes: [per-frame-token-stream]
produces: [cross-frame-attended-tokens]
invariants: []
provides_properties: []
requires_upstream_properties: [vit-per-frame-tokenization]
data_regime: [offline-or-streaming]
tags: [attention]
created: 2026-04-18
updated: 2026-04-24
---

Umbrella stage for the cross-frame attention pattern that aggregates information across frames in a feed-forward SfM model. Previously flagged as alias of [[feed-forward-sfm.global-attention]]; split effective 2026-04-24 after LingBot-Map demonstrated that `global-attention` is only *one* specialization of the stage.

Distinct filler families:

- **Full global attention** → [[feed-forward-sfm.global-attention]] — every frame attends to every other frame with full tokens. VGGT-family.
- **Causal attention** — Stream3R, StreamVGGT, Wint3R. Linear-memory growth streaming variant of global attention.
- **TTT fast-weights** — [[feed-forward-sfm.recurrent-state-update]]. Replaces attention-function-class with recurrent fast-weights (LoGeR, ZipMap, TTT3R).
- **Structured three-context attention** → [[three-context-geometric-attention_chen2026]]. Partitions attention keys into typed contexts (anchor / pose-window / trajectory memory) grounded in classical SLAM's architecture. LingBot-Map.

When a new filler introduces a new typed context or eviction policy, file as a new idea and co_require this stage plus any new sub-stage it creates (e.g., LingBot-Map also introduces [[feed-forward-sfm.coordinate-grounding]]).
