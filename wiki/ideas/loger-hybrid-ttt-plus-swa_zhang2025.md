---
title: Hybrid TTT + sliding-window attention for 19K-frame reconstruction (LoGeR)
type: idea
source_paper: wiki/papers/zhang2025_loger.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.long-context-memory]
inputs: [multi-chunk-token-stream]
outputs: [global-compressed-context-plus-lossless-local-context]
assumptions: [chunk-wise-processing-feasible, pi3-backbone-available, training-budget-for-ttt-plus-swa]
requires_upstream_property: [pi3-or-vggt-style-backbone]
requires_downstream_property: [prediction-heads-accept-hybrid-memory-output]
learned_params: [fast-weights-per-chunk, swa-layer-weights, curriculum-trained]
failure_modes: [training-cost-higher-than-ttt3r-training-free]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, ttt, sliding-window-attention, hybrid-memory]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Hybrid memory**: parametric TTT fast weights `W^m` compress global context; non-parametric sliding-window attention (SWA) carries lossless local context across adjacent chunks. **Chunk-wise TTT**: muon-style fast-weight updates per chunk anchor a global coordinate frame, preventing scale drift on thousands of frames. **Curriculum training** (48 → 128 frames, TartanAirV2 heavily weighted) addresses the data wall alongside the context wall.

## Why it wins

19K-frame / 11.5km reconstruction without post-optimization. ATE reduction 32.5% over VGGT-Long. Lossless local context (SWA) + lossy global compression (TTT) gives both precision-locally and scale-globally.

## Preconditions & compatibility

Training-cost higher than training-free TTT3R. Bet: combine LoGeR's SWA leg with TTT3R's training-free closed-form LR for the best-of-both-worlds configuration. No paper does this yet.
