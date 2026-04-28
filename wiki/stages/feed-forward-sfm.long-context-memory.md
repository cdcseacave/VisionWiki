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
tags: [loger, ttt, sliding-window, token-eviction]
created: 2026-04-18
updated: 2026-04-24
---

Long-range memory mechanism for scaling feed-forward SfM to thousands of frames. Family structure (three competing paradigms):

- **Hybrid TTT + SWA** — [[loger-hybrid-ttt-plus-swa_zhang2025]] (LoGeR). Non-parametric sliding window + parametric TTT state.
- **Pure TTT fast-weights** — [[large-chunk-ttt-fast-weights_jin2026]] (ZipMap), [[ttt3r-closed-form-confidence-lr_chen2026]] (TTT3R). Replaces memory with inference-time parameter updates.
- **Compact-token eviction + temporal positional encoding** — [[compact-trajectory-memory-tokens_chen2026]] + [[video-rope-on-trajectory-tokens_chen2026]] (LingBot-Map GCA). Retains 6 tokens/frame for evicted frames; Video RoPE injects temporal order.

Orthogonal systems-level optimization applicable to any family: [[paged-kv-cache-streaming-sfm_chen2026]] (paged KV-cache for evict/append churn, ~2× inference speedup).
