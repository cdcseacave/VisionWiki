---
title: Human/model-in-the-loop data engine for real-image 3D annotation
type: idea
source_paper: wiki/papers/chen2025_sam-3d.md
also_in: []

scope: drop-in
stages: [generative-3d.training-data-curation]
inputs: [synthetic-3d-pretraining, real-image-annotations]
outputs: [large-scale-real-image-3d-training-set]
assumptions: [annotator-budget-available, synthetic-pretraining-feasible]
requires_upstream_property: [base-3d-dataset-for-pretraining]
requires_downstream_property: [consumer-is-generative-or-regression-model]
learned_params: []
failure_modes: [3d-data-barrier-remains-without-sustained-annotation-budget]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [generative-3d, data-engine, single-image]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Combine synthetic 3D pretraining with real-image alignment via a human+model data engine: model makes proposals on real images, humans correct, proposals are re-ingested as training data. Breaks the chicken-and-egg problem of "no real 3D annotations → no generalization → need real 3D annotations."

## Why it wins

The data-side contribution that actually enables generalization from isolated synthetic objects to cluttered natural images. Without this, Stage-1 generators train fine on synthetic but collapse on real images.

## Preconditions & compatibility

Bundles with the shape generator (`co_requires:` on the main idea). Generalizable pattern beyond SAM 3D — any generative 3D model facing the data barrier can adopt the recipe.

## Open questions

- How many humans × hours does it take to produce a useful training distribution?
- Does the recipe work for dynamic scenes?
