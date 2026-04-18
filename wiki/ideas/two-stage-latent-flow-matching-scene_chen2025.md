---
title: Two-stage latent flow-matching for single-image 3D scene (SAM 3D)
type: idea
source_paper: wiki/papers/chen2025_sam-3d.md
also_in: []

scope: new-paradigm
stages: [generative-3d.shape-generation, generative-3d.scene-layout]
collapses: []
splits_into: []
rewrites: {}

inputs: [single-rgb-image]
outputs: [per-object-geometry, per-object-texture, scene-layout-placement]
assumptions: [single-image-input, cluttered-natural-image-domain, commercial-license-blocked-sam-license]
requires_upstream_property: [per-object-mask-from-sam3-or-user]
requires_downstream_property: [consumer-composites-objects-into-scene]
learned_params: [shape-generator-weights, layout-module-weights]
failure_modes: [generative-hallucination-vs-fidelity-underconstrained, no-lighting-consistency-with-existing-scene]

requires: []
unlocks: []
co_requires: [real-image-data-engine_chen2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [generative-3d, single-image, flow-matching, meta-sam-3d]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Stage 1 — **shape generator**: image features → latent → decoded geometry + texture per object via flow-matching. Stage 2 — **layout module**: predicts placement (pose + scale) of each generated object in the scene coordinate frame. The "two-stage" formulation decouples per-object synthesis from inter-object composition, so each stage can be trained with its own loss and data distribution.

## Why it wins

≥5:1 human preference win rate vs prior single-image 3D SOTA on real-world objects + scenes. Extends from isolated-object generators (which dominated 2023-2024) to *cluttered* natural images. Orthogonal to DUSt3R/MASt3R/VGGT — those need ≥2 views; SAM 3D operates from a single image.

## Preconditions & compatibility

SAM License (non-commercial research only). Bundles with its data engine (`co_requires:`) — without the real-image alignment data, the generator generalizes poorly from synthetic-only pretraining. Natural downstream composition: SAM 3 masks as input conditioning (naming suggests this but paper doesn't demonstrate).

## Pipeline-shape implications

New-paradigm: lives in a parallel pipeline from the reconstruction-based threads. A thread adopting this cannot also use DUSt3R-style pointmap prediction as its primary stage — the two operate under different input assumptions.

## Open questions

- Reconstruction-vs-generation trade not quantified.
- Compositing into existing scenes (texture + lighting consistency) untreated.
