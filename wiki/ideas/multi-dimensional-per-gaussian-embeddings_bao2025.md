---
title: Multi-dimensional per-Gaussian embeddings (identity + appearance + lighting-invariant)
type: idea
source_paper: wiki/papers/bao2025_seg-wild.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.per-primitive-identity]
inputs: [3d-gaussians, multi-view-images, per-image-metadata]
outputs: [multi-channel-per-gaussian-embedding]
assumptions: [in-the-wild-capture, transient-occluders-present, lighting-drift-between-images]
requires_upstream_property: [per-image-context-available]
requires_downstream_property: [renderer-emits-multi-channel-output]
learned_params: [per-gaussian-identity, per-gaussian-appearance, per-gaussian-lighting-invariant, per-image-embedding]
failure_modes: [sgc-threshold-scene-dependent, interactive-query-still-user-driven]

requires: []
unlocks: []
co_requires: [spiky-3d-gaussian-cutter_bao2025]
bridges: []
equivalent_to: []
refines: [gaussian-grouping-identity-encoding_ye2024]
contradicts: []

tags: [3dgs, in-the-wild, identity-embedding, multi-channel]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Extend Gaussian Grouping's single identity vector to **three separate channels** per Gaussian: identity, appearance (per-image transient component), and lighting-invariant. Each channel learns independently; the explicit separation disambiguates transient-occluded pixels that would otherwise corrupt identity supervision.

## Why it wins

Refines [[gaussian-grouping-identity-encoding_ye2024]] for in-the-wild captures (internet photo collections) where appearance drift + transient occluders would pollute a single identity channel. The per-image embedding decouples geometry from photometric variation.

## Preconditions & compatibility

Bundles with the Spiky 3D Gaussian Cutter (`co_requires:`) — without SGC, Gaussians crossing SAM mask boundaries still leak identity across instances.

## Open questions

- SGC threshold is hand-tuned; scene-dependent.
- Natural composition with VastGaussian's decoupled appearance for city-scale in-the-wild segmentation — untried.
