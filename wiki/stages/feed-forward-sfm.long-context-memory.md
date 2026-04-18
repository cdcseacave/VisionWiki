---
title: Feed-forward SfM long-context memory
type: stage
slug: feed-forward-sfm.long-context-memory
consumes: [chunked-token-stream]
produces: [compressed-global-context-plus-local-window-context]
invariants: [global-coordinate-frame-anchored]
provides_properties: [scale-drift-bounded-over-thousands-of-frames]
requires_upstream_properties: [tokenizer-and-per-chunk-processing-available]
data_regime: [long-sequences-100-to-20k-frames]
tags: [loger, ttt, sliding-window]
created: 2026-04-18
updated: 2026-04-18
---

Hybrid parametric TTT + non-parametric SWA memory (LoGeR) or pure TTT (ZipMap) for scaling to thousands of frames. Example fillers: [[loger-hybrid-ttt-plus-swa_zhang2025]], [[large-chunk-ttt-fast-weights_jin2026]].
