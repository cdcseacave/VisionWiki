---
title: Differentiable plane-sweep cost volume with variance aggregation (MVSNet)
type: idea
source_paper: wiki/papers/yao2018_mvsnet.md
also_in: []

scope: new-paradigm
stages: [mvs.feature-aggregation, mvs.depth-regression, mvs.depth-refinement]
collapses: []
splits_into: []
rewrites: {}

inputs: [reference-image, n-source-images, per-image-features]
outputs: [per-reference-pixel-depth-map]
assumptions: [known-poses, moderate-baseline, lambertian-default]
requires_upstream_property: [cnn-or-vit-image-encoder]
requires_downstream_property: [consumer-uses-dense-depth-map]
learned_params: [feature-extractor-weights, 3d-cnn-weights, refinement-head-weights]
failure_modes: [memory-scales-with-depth-hypothesis-count, transformer-based-descendants-dominate-at-high-end]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mvs, cost-volume, learned-mvs, foundational]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

N warped feature volumes reduced to one by per-voxel **variance** across N — a permutation-invariant aggregation. 3D-CNN regularizes the cost volume; soft-argmin produces subpixel depth via expected-depth regression. Color-guided refinement uses the reference image to produce an edge-aligned residual correction.

## Why it wins

GPU-friendly, arbitrary-N inputs, subpixel depth. The template every learned-MVS paper inherits. Proved "learned MVS > classical PatchMatch" on DTU and T&T. Its cost-volume template still appears in feed-forward 3DGS papers (MVSplat, MVSGaussian).

## Preconditions & compatibility

Conceptual ancestor of DUSt3R / MASt3R / VGGT — pointmap methods replace the explicit cost volume with a transformer, but MVSNet remains the reference for "arbitrary-N feature aggregation in a learned MVS."

## Open questions

- Does variance-aggregation still dominate vs. attention-based aggregation at large N?
