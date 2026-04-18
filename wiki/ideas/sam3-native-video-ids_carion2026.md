---
title: Promptable Concept Segmentation with native cross-view IDs (SAM 3)
type: idea
source_paper: wiki/papers/carion2026_sam-3.md
also_in: []

scope: multi-stage-collapse
stages: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries, open-vocab-2d.cross-frame-tracking]
collapses: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries, open-vocab-2d.cross-frame-tracking]
splits_into: []
rewrites: {}

inputs: [image-or-video, text-noun-phrase-or-image-exemplars]
outputs: [per-concept-masks, per-instance-ids, video-tracks]
assumptions: [commercial-license-ok-NO-sam-license-is-research-only, concept-is-noun-phrase-or-exemplar, moderate-occlusion-budget]
requires_upstream_property: [none-sam3-is-a-frontend]
requires_downstream_property: [consumer-accepts-mask+id+track-output]
learned_params: [full-sam3-weights]
failure_modes: [compositional-prompts-fail, long-occlusion-id-drift]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: [cross-view-mask-association-iou-similarity_ye2024]

tags: [sam, concept-segmentation, video-tracking, 2d-foundation-model]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

A single vision backbone feeds two branches: (a) an image-level detector producing per-frame masks for the prompted concept; (b) a memory-based video tracker propagating instance IDs across frames. A **presence head** separately predicts "is the concept in the image at all" — decoupling false-positive suppression from per-instance localization. Prompts are either text noun phrases (via a text encoder) or image exemplars (via the vision encoder); both routes hit the same PCS head. Trained on SA-Co (4M concept labels with hard-negative mining).

## Trade-offs vs. the decomposed pipeline

Pipelines that use CLIP (semantic) + DINO (spatial) + SAM (mask) + a separate tracker lose when they are fused because per-stage interpretability disappears: if SAM 3 is wrong, there is no "CLIP stage" to introspect. Mitigation: the presence head still exposes a per-prompt confidence. A pre-migration pipeline that wanted to inject a custom spatial-coherence prior at the DINO stage can't do so in SAM 3; the only knob is the prompt. Trade favorable when inference latency or cross-frame tracking dominates; unfavorable when modular debugging does.

## Why it wins

2× accuracy on image + video PCS benchmarks vs. prior SOTA. Presence head removes "hallucinated detections on absent concepts" — a consistent Trident/GroundingDINO failure. Video tracking comes for free rather than being bolted on. Single forward pass replaces three.

## Preconditions & compatibility

SAM License is non-commercial research-only — materially restricts downstream use. Contradicts heuristic cross-view association schemes (e.g. [[cross-view-mask-association-iou-similarity_ye2024]]) — one canonical downstream move is to *delete* those modules when SAM 3 is the mask source.

## Pipeline-shape implications

Collapses four stages in `open-vocab-2d` (semantic-alignment, spatial-coherence, mask-boundaries, cross-frame-tracking) into one composite node. Any thread adopting SAM 3 must represent the node as composite.

## Open questions

- Compositional prompts ("red chair *without* armrests") untested.
- 3D lifting is left for downstream work — Bets #013 (Gaussian Grouping + SAM 3) and #020 (language-grounded 3DGS 2026 stack) are the canonical downstream synthesis bets.
