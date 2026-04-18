---
title: Closed-form confidence-guided per-token learning rate (TTT3R)
type: idea
source_paper: wiki/papers/chen2026_ttt3r.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.recurrent-state-update]
inputs: [frozen-backbone-attention-QK, current-state, new-observation]
outputs: [per-token-learning-rate, updated-state]
assumptions: [cut3r-style-recurrent-state, frozen-backbone, no-retraining-budget]
requires_upstream_property: [dino-or-croco-tokenized-attention-available]
requires_downstream_property: [recurrent-update-accepts-per-token-lr]
learned_params: []
failure_modes: [ood-regime-on-very-long-rollouts-needs-state-reset, doesnt-close-gap-with-full-attention]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, test-time-training, frozen-backbone, cut3r, training-free]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Reinterpret CUT3R's recurrent state as a fast weight. The per-token learning rate is derived *closed-form* from the existing frozen-backbone attention: `β_t = σ(Σ_m Q_{S_{t-1}} K_{X_t}^⊤)`. High alignment between state queries and observation keys → low learning rate (state is already good), low alignment → high learning rate (state needs to absorb). No new weights, no retraining. Optional State Reset + SE(3) chunk alignment prevents fast-weight saturation on >1K frame rollouts.

## Why it wins

2× pose improvement on long sequences over CUT3R; runs at 20 FPS / 6 GB for thousands of images. Training-free — LoGeR and ZipMap require training new models for the same benefit. The closed-form derivation *from the existing attention* is the load-bearing insight.

## Preconditions & compatibility

Doesn't close the gap with full-attention (VGGT) when memory permits; TTT3R mitigates forgetting but doesn't eliminate it. Bet #012 applies the same closed-form LR derivation to RoMa v2's dense matching head — the pattern generalizes.

## Open questions

- Transfer to non-pointmap feed-forward models (e.g. feed-forward radiance-field methods)?
- Interaction with downstream BA refinement — does TTT3R absorb what BA would add, or leave room?
