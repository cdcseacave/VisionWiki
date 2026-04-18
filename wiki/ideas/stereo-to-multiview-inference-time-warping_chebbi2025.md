---
title: Train-stereo-test-multiview via inference-time geometric priors
type: idea
source_paper: wiki/papers/chebbi2025_multiview-dense-matching.md
also_in: []

scope: stage-swap
stages: [mvs.feature-aggregation]
inputs: [stereo-trained-features, n-source-images, known-poses]
outputs: [plane-swept-multi-view-depth-from-stereo-features]
assumptions: [only-stereo-training-data-available, inference-poses-known]
requires_upstream_property: [stereo-trained-feature-extractor]
requires_downstream_property: [consumer-uses-multi-view-depth-map]
learned_params: [stereo-cnn-weights, ms-aff-backbone]
failure_modes: [large-non-rectifiable-baselines]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mvs, stereo-transfer, aerial, lightweight]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Train feature extractor on epipolar stereo pairs (plenty of stereo GT). At inference, warp features into **rotation-aligned geometries** (epipolar or homographic) via known poses, then plane-sweep for arbitrary N views. The "multi-view" part is inference-time warping, not learned multi-view aggregation. Lightweight MS-AFF backbone (8–10× lighter than U-Net variants) + multi-resolution cascade that uses NCC at coarse levels + learned features at high-res.

## Why it wins

Matches end-to-end learned MVS in-distribution; **generalizes OOD** where PSMNet / MVSNet collapse. No multi-view GT needed. Natural aerial/satellite MVS depth source for Pipeline A in [[gaussian-to-mesh-pipelines]] and [[radiance-field-evolution]]'s city-scale OP.

## Open questions

- Performance at very large baselines where rotation-alignment breaks?
