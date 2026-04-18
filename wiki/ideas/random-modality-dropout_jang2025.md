---
title: Random modality dropout training for arbitrary prior combinations
type: idea
source_paper: wiki/papers/jang2025_pow3r.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.training-recipe]
inputs: [multi-modal-training-data]
outputs: [single-model-handling-all-prior-subsets]
assumptions: [training-budget-allows-modality-dropout-schedule]
requires_upstream_property: []
requires_downstream_property: [inference-time-accepts-any-prior-subset]
learned_params: []
failure_modes: [dropout-schedule-hyperparameter-tuning-needed]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [training-recipe, modality-dropout, prior-injection]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Randomly drop each input modality (intrinsics, poses, depth) with some probability during training. At inference, any subset can be supplied — the model learns to condition on whichever priors are present and ignore whichever are absent.

## Why it wins

One model, all inference configurations — no per-setup fine-tuning. Generalizable pattern beyond Pow3R: any model with optional auxiliary inputs can adopt the recipe.

## Open questions

- Can the dropout probability be learned rather than hyperparameter?
