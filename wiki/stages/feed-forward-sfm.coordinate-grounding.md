---
title: Feed-forward SfM coordinate grounding
type: stage
slug: feed-forward-sfm.coordinate-grounding
consumes: [raw-image-stream-or-token-stream]
produces: [reference-frame-plus-scale-anchor]
invariants: [coordinate-origin-fixed, scale-factor-defined]
provides_properties: [canonical-coordinate-frame, metric-or-up-to-scale-reference]
requires_upstream_properties: [tokenizer-or-backbone-features]
data_regime: [streaming-or-offline, monocular-or-multi-view]
tags: [scale-grounding, coordinate-system, gauge-decoupling]
created: 2026-04-24
updated: 2026-04-24
---

Stage for how a feed-forward SfM model establishes its global coordinate frame and scale. Every feed-forward method has an implicit or explicit mechanism here — historically treated as an architectural detail, but Wang 2026 §4.5.1 (gauge-decoupled streaming) flagged it as a first-class scaling concern for streaming models.

Prior-art fillers (implicit):
- [[dust3r|DUSt3R]] / [[leroy2024_mast3r|MASt3R]] — two-view pair-pointmap alignment defines the scale (pair-wise metric via [[metric-scale-pointmap-loss_leroy2024]] for the metric variant).
- [[vggt|VGGT]] — global point-cloud normalization across all input frames; incompatible with causal streaming because it requires all frames.
- Stream3R / StreamVGGT — inherits VGGT normalization but compromises on the causal-stream setting.

Explicit filler:
- [[anchor-frame-scale-grounding_chen2026]] — designates first $n$ frames as anchor; normalizes by anchor point-cloud mean distance. Compatible with streaming by construction.

Notes on valid fillers: must produce a *consistent* reference-frame + scale that downstream stages (attention-mechanism, long-context-memory) can condition on. Failure modes include degenerate geometry (static first frames → ill-defined scale) and drift (continuously-updated reference → gauge not decoupled from trajectory).
