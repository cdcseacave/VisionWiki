---
title: Feed-forward SfM training recipe
type: stage
slug: feed-forward-sfm.training-recipe
consumes: [multi-modal-training-data]
produces: [single-model-handling-diverse-inference-configs]
invariants: [generalization-across-prior-subsets]
provides_properties: [modality-dropout-or-curriculum]
requires_upstream_properties: [training-data-with-prior-labels]
data_regime: [training-time]
tags: [training-recipe, modality-dropout, curriculum]
created: 2026-04-18
updated: 2026-04-18
---

How the model is trained for robust inference-time behavior. Random modality dropout (Pow3R) enables arbitrary-subset inference; curriculum (LoGeR 48→128 frames) addresses the data wall. Example fillers: [[random-modality-dropout_jang2025]], [[loger-hybrid-ttt-plus-swa_zhang2025]].
