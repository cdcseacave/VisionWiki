---
title: SAM mask-hierarchy supervision (whole / part / subpart) for CLIP lifting
type: idea
source_paper: wiki/papers/qin2024_langsplat.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.multi-scale-semantic-supervision]
collapses: []
splits_into: []
rewrites: {}

inputs: [per-view-images, sam-mask-hierarchy-per-view, per-mask-clip-embedding]
outputs: [per-scale-supervised-language-field]
assumptions: [sam-hierarchy-semantically-meaningful, scale-of-query-matches-scale-of-training]
requires_upstream_property: [sam-mask-hierarchy-available]
requires_downstream_property: [language-field-is-per-primitive-and-scale-aware]
learned_params: []
failure_modes: [sam-failure-on-thin-or-transparent-objects, hierarchy-levels-don't-match-user-query-granularity]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, sam, clip, multi-scale]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

SAM outputs three nested masks per click: whole, part, subpart. For each of these levels, crop the corresponding region, pass through CLIP, and get a mask-level CLIP embedding at that scale. Train the per-Gaussian language field to match the CLIP embedding at *all three scales*. Query time: a single forward pass produces the rendered language-field at multiple scales simultaneously — no image-scale sweeps needed.

## Why it wins

Baked-in multi-scale supervision replaces LERF's expensive query-time multi-scale rendering. Hierarchy also gives the language field structural priors: "subpart features must be consistent with their containing part" — which improves boundary sharpness without explicit DINO regularization.

## Preconditions & compatibility

Requires SAM or any hierarchical mask source (SAM 3 also emits hierarchies). Compatible with any per-primitive feature-storage scheme (bundles with LangSplat's autoencoder but also works with raw CLIP features if memory allows).

## Open questions

- SAM 3's concept-driven masks may be less hierarchical than SAM 1's whole/part/subpart — compatibility needs testing.
- Query-time scale ambiguity: user says "chair" — which of whole/part/subpart matches?
