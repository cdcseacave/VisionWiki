---
title: Large-scale synthetic indoor dataset (12k scenes / 55k rooms) for SpatialLM
type: idea
source_paper: wiki/papers/mao2025_spatiallm.md
also_in: []

scope: drop-in
stages: [llm-structured-scenes.pretraining-data]
inputs: [synthetic-scene-generator]
outputs: [scene-annotation-pairs-for-pretraining]
assumptions: [synthetic-annotations-transferable-to-real-scans]
requires_upstream_property: []
requires_downstream_property: [consumer-fine-tunes-on-small-real-dataset]
learned_params: []
failure_modes: [cross-domain-gap-synth-to-real-remains]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [dataset, synthetic, indoor, spatiallm]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Procedurally generated synthetic indoor scenes (12,328 scenes, 54,778 rooms) with ground-truth walls, doors, windows, oriented bboxes. Used as pretraining data; a small real-scan dataset fine-tunes afterward.

## Why it wins

Paper's ablation: without the synthetic pretraining, fine-tuning on small real datasets only → performance collapses. The synthetic corpus closes the 3D data gap that every specialist layout model struggles with.

## Open questions

- Does the dataset help non-LLM architectures too? Not ablated.
