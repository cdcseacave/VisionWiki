---
title: Feed-forward SfM global attention
type: stage
slug: feed-forward-sfm.global-attention
consumes: [per-frame-tokens]
produces: [attention-aggregated-tokens]
invariants: [cross-frame-information-flow-respected]
provides_properties: [o-n-or-o-n2-scaling-depending-on-filler]
requires_upstream_properties: [tokenizer-output]
data_regime: [multi-view-transformer]
tags: [vggt, fast3r, pointmap-models]
created: 2026-04-18
updated: 2026-04-18
---

Cross-frame attention layer. Quadratic softmax (VGGT) vs. linear-cost alternatives (ZipMap TTT fast weights, VGG-T3 KV compression). Example fillers: [[large-chunk-ttt-fast-weights_jin2026]], [[ttt-mlp-kv-compression_elflein2026]].
