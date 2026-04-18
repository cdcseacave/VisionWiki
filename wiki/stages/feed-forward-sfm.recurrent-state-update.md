---
title: Feed-forward SfM recurrent-state update
type: stage
slug: feed-forward-sfm.recurrent-state-update
consumes: [prior-state, new-observation-tokens, frozen-backbone-attention]
produces: [updated-state]
invariants: [state-size-bounded-regardless-of-sequence-length]
provides_properties: [long-context-scaling-linear]
requires_upstream_properties: [frozen-tokenizer]
data_regime: [long-sequence-multi-view]
tags: [cut3r, ttt3r, recurrent, pointmap]
created: 2026-04-18
updated: 2026-04-18
---

How the recurrent state absorbs a new observation. Closed-form confidence-guided learning rate (TTT3R) derives the per-token update rate from existing attention. Example fillers: [[ttt3r-closed-form-confidence-lr_chen2026]].
