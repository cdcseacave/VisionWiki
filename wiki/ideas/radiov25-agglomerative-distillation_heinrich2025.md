---
title: Multi-resolution agglomerative distillation from SigLIP + DINOv2 + SAM
type: idea
source_paper: wiki/papers/heinrich2025_radiov25.md
also_in: []

scope: multi-stage-collapse
stages: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries]
collapses: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries]
splits_into: []
rewrites: {}

inputs: [training-images, siglip-teacher, dinov2-teacher, sam-teacher]
outputs: [unified-student-backbone]
assumptions: [teacher-quality-is-the-ceiling, multi-res-training-budget-available, commercial-license-blocked-NSCL]
requires_upstream_property: [all-teachers-accessible]
requires_downstream_property: [consumer-uses-single-backbone-output]
learned_params: [student-backbone-weights]
failure_modes: [capability-frozen-at-distillation-time, teacher-quality-ceiling]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [foundation-model, agglomerative-distillation, multi-resolution, unified-backbone]
created: 2026-04-18
updated: 2026-04-22
status: unclaimed
validated_in: []
---

> **Refined by**: [[cradiov4-agglomerative-distillation_ranzinger2026]] (2026-01-27).
> C-RADIOv4 upgrades the teacher set (SigLIP2/DINOv3/SAM3), adds stochastic
> resolutions, shift-equivariant distillation loss + MESA, angle-normalized
> summary loss, ViTDet deployment mode, and releases under the NVIDIA Open
> Model License (commercial use permitted — resolves this idea's
> `commercial-license-blocked-NSCL` assumption). For new adoption, prefer
> C-RADIOv4 unless you specifically need RADIOv2.5's weights. The RADIOv2.5
> mechanism below remains the canonical reference for the core
> agglomerative-distillation pattern.


## Mechanism

Train a single ViT student at multiple resolutions simultaneously against three teacher backbones (SigLIP semantic, DINOv2 spatial, SAM mask) using MSE feature-distillation + cosine summary loss. PHI-S normalizes teacher distributions so no single teacher dominates the loss. Token merging (ToMe) compresses the student's token budget at inference without losing fine-grained info. The multi-resolution aspect is critical — it kills the "mode switch" where DINO-like features live at low-res and SAM-like features at high-res, a failure mode of the baseline AM-RADIO.

## Trade-offs vs. the decomposed pipeline

Trident's three separate backbones let you swap one component (e.g. CLIP → SigLIP) without retraining. RADIOv2.5 fixes the capability at distillation time — if a new CLIP lands, you retrain the student. You gain single-pass inference (3× faster than running all three separately) and lose modularity. Trade favorable when inference budget dominates; unfavorable when a new frontier teacher would obsolete the student.

## Why it wins

Ablations show each component (multi-res, SigLIP teacher, ViT-H scale, ToMe) contributes. Probe3D gives depth 85.7% + surface-normal 62.5%; ADE20k 53.97% mIoU; VLM VILA on TextVQA jumps from 57.01% → 69.74%. Multi-resolution training dramatically reduces scale-equivariance variance vs AM-RADIO.

## Preconditions & compatibility

NVIDIA Source Code License (non-commercial research only) — blocks commercial integration. Requires all teachers accessible at training time; cannot distill from a future closed teacher. Natural downstream composition: RADIOv2.5 as the per-Gaussian feature source in LangSplat (Bet #014).

## Pipeline-shape implications

Collapses three Trident stages into one student backbone. Threads using Pipeline C (distilled-single-pass) must represent this as a composite node spanning all three semantic/spatial/mask stages.

## Open questions

- Optimal teacher count and selection not explored.
- ToMe compression ratio vs. information loss needs task-specific tuning.
