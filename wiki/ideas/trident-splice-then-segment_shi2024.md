---
title: Trident Splice-then-Segment with SAM affinity + triple-prompt refinement
type: idea
source_paper: wiki/papers/shi2024_open-vocab-segmentation.md
also_in: []

scope: topology-rewrite
stages: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries, open-vocab-2d.aggregation]
collapses: []
splits_into: []
rewrites: {replaces: [open-vocab-2d.aggregation], introduces: [open-vocab-2d.splice-then-segment-aggregation]}

inputs: [image, text-prompt, frozen-clip, frozen-dinov3, frozen-sam]
outputs: [open-vocab-segmentation-mask]
assumptions: [three-frozen-backbones-available, inference-budget-allows-three-forward-passes]
requires_upstream_property: [clip-dino-sam-all-available]
requires_downstream_property: [consumer-uses-final-mask]
learned_params: []
failure_modes: [three-ViT-cost, sam-features-over-segment-object-subparts, affinity-threshold-epsilon-sensitive]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [open-vocab, training-free, trident, clip-dino-sam]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Splice-then-Segment** inverts the older Segment-then-Splice paradigm. (1) Splice: CLIP features are spliced across sub-images, aggregated via a SAM affinity matrix (cross-window attention, `A = (W+M) / ||W+M||` from SAM encoder features). (2) Segment: the aggregated feature is classified against text prompts to produce a coarse map, which then prompts SAM with triple inputs (point + box + mask) for boundary refinement.

## Why it wins

+4.2% mIoU over prior training-free SOTA across 8 benchmarks. Resolves the resolution-vs-receptive-field scaling trap of sliding-window Segment-then-Splice (where receptive fields shrink with window size). SAM's encoder features carry large-scale coherence that DINO's patch-level SSL misses — combining both covers different spatial scales.

## Preconditions & compatibility

Three-ViT inference cost. SAM's features capture low-level visual similarity rather than semantic relationships — over-segments objects with distinct sub-parts (the acknowledged failure mode). Bet #017 upgrades CLIP → SigLIP, DINO → DINOv3, SAM → SAM 3.

## Pipeline-shape implications

Topology-rewrite: defines the four-stage open-vocab pipeline that subsequent papers refine one leg at a time.
