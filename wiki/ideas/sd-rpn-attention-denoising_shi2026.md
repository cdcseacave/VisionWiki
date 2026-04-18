---
title: Self-distilled attention denoising via sink-token removal (SD-RPN)
type: idea
source_paper: wiki/papers/shi2026_self-distilled-roi.md
also_in: []

scope: drop-in
stages: [open-vocab-2d.spatial-coherence]
inputs: [frozen-backbone-attention-map, l2-norm-threshold]
outputs: [denoised-attention-map, foreground-background-pseudo-labels]
assumptions: [backbone-has-sink-tokens, ambiguous-tokens-can-be-excluded]
requires_upstream_property: [any-frozen-backbone-exposing-attention]
requires_downstream_property: [consumer-accepts-pseudo-label-supervision]
learned_params: []
failure_modes: [fixed-thresholds-per-backbone, ambiguous-regions-excluded-lose-signal]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [attention-denoising, self-distillation, mllm, sink-tokens]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Identify high-L2-norm "sink tokens" in the MLLM's cross-attention map (these absorb attention mass without carrying spatial information). Remove them from the attention map to produce a clean signal. Apply selective foreground/background pseudo-labeling: tokens with attention `a_j ≥ τ_fg · a_max` are foreground, `a_j ≤ τ_bg · a_max` are background, everything in between is excluded as ambiguous. Use the resulting pseudo-labels to train an RPN without external supervision.

## Why it wins

+10–16% on TextVQA/DocVQA/V-Star with only 10K QA pairs; 2.5× faster than ViCrop. The sink-token removal is the load-bearing insight — without it the RPN trains on noise. Generalizable: the **attention-as-spatial-signal pattern** works for any frozen backbone that exposes attention (DINO, CLIP, SAM prompt-distillation, potentially 3DGS render-distillation).

## Preconditions & compatibility

Fixed thresholds (`τ_fg`, `τ_bg`, `τ_norm`) may need per-backbone tuning. Bet #019 applies the same denoising recipe to DINOv3 self-attention for Trident's spatial-coherence stage.

## Open questions

- RoIs processed independently — multi-RoI spatial relationships lost.
- Does the sink-token phenomenon appear in non-transformer backbones (SSMs, convs)?
