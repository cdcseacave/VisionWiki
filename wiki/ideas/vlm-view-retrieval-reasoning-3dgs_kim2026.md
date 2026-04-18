---
title: Query-driven NVS retrieval + VLM reasoning on 3DGS (GaussExplorer)
type: idea
source_paper: wiki/papers/kim2026_gauss-explorer.md
also_in: []

scope: new-paradigm
stages: [vlm-reasoning.view-retrieval, vlm-reasoning.novel-view-adjustment, vlm-reasoning.compositional-answering]
collapses: []
splits_into: []
rewrites: {}

inputs: [trained-3dgs-scene, natural-language-query, pretrained-vlm]
outputs: [answer-to-compositional-query, evidence-views]
assumptions: [3dgs-already-trained-per-scene, vlm-available-at-inference, nvs-quality-sufficient-to-not-fool-vlm]
requires_upstream_property: [3dgs-trained-on-the-target-scene]
requires_downstream_property: [consumer-accepts-vlm-textual-answer]
learned_params: []
failure_modes: [vlm-inference-dominates-runtime-not-realtime, nvs-artifacts-hallucinated-as-scene-content]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, vlm, reasoning, inference-time-composition]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Given a 3DGS scene + a natural-language query: (1) top-K retrieve pre-captured views most likely to answer the query; (2) iteratively render novel views in the neighborhood of the retrieved views, scored by information gain; (3) hand the composite view set to an off-the-shelf VLM for compositional reasoning. No per-Gaussian feature lifting; 3DGS is used as a queryable scene database, not as a carrier of semantic features.

## Why it wins

Compositional-reasoning benchmarks where single-view or feature-lifted methods fail (questions that need to compare multiple parts of the scene). The "consumer side of lifted 3D" — doesn't train per-Gaussian features, consumes them via rendered evidence instead. Wins where LangSplat/Gaussian Grouping's per-scene distillation can't keep up with free-form compositional queries.

## Trade-offs vs. the decomposed pipeline

Feature-lifted pipelines (LangSplat et al.) answer queries in milliseconds via per-Gaussian cosine lookup; GaussExplorer takes seconds because the VLM forward pass dominates. Gain: no per-scene feature-distillation cost, no fixed vocabulary. Loss: real-time is off the table.

## Pipeline-shape implications

New-paradigm: the query pipeline is retrieve → render → reason, not encode → lookup. Threads adopting this split into a parallel pipeline (can't mix with feature-lifted flow without duplication).

## Open questions

- Evaluated on compositional QA; embodied planning untested.
- Bridge to SAM 3 exemplar prompts (which also use renderable evidence) unexplored.
