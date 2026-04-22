---
title: C-RADIOv4 refined multi-resolution agglomerative distillation (SigLIP2 + DINOv3 + SAM3)
type: idea
source_paper: wiki/papers/ranzinger2026_c-radiov4.md
also_in: []

scope: multi-stage-collapse
stages: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries]
collapses: [open-vocab-2d.semantic-alignment, open-vocab-2d.spatial-coherence, open-vocab-2d.mask-boundaries]
splits_into: []
rewrites: {}

inputs: [training-images, siglip2-teacher, dinov3-teacher, sam3-teacher]
outputs: [unified-student-backbone, sam3-compatible-adaptor]
assumptions: [teacher-quality-is-the-ceiling, teacher-positional-noise-present, commercial-license-OK, any-resolution-support, patch-size-aligned-shifts-available]
requires_upstream_property: [all-teachers-accessible]
requires_downstream_property: [consumer-uses-single-backbone-output]
learned_params: [student-backbone-weights, per-teacher-adaptor-heads]
failure_modes: [capability-frozen-at-distillation-time, sam3-replacement-lossy-on-domain-specific-queries, teacher-angular-dispersion-must-be-estimable, no-ablation-isolating-each-training-side-change]

requires: []
unlocks: []
co_requires: [shift-equivariant-distillation-loss_ranzinger2026]
bridges: []
equivalent_to: []
refines: [radiov25-agglomerative-distillation_heinrich2025]
contradicts: []

tags: [foundation-model, agglomerative-distillation, multi-resolution, unified-backbone, commercial-license, vitdet, siglip2, dinov3, sam3]
created: 2026-04-22
updated: 2026-04-22
status: unclaimed
validated_in: []
---

## Mechanism

Train a single ViT student at stochastically-sampled resolutions (from {128, 192, 224, 256, 384, 432} low-res and {512, 768, 1024, 1152} high-res partitions, per-batch uniform sample) against three teacher backbones (SigLIP2-g-384 semantic, DINOv3-7B spatial, SAM3 mask). Four training-side refinements over [[radiov25-agglomerative-distillation_heinrich2025|RADIOv2.5]]:

1. **Stochastic resolutions** replace the two-resolution schedule. For fixed-resolution teachers (SigLIP2 @ 384px), FeatSharp upsamples teacher features 3× to match the student's high-res grid. SAM3 (fixed 1152²) uses mosaic augmentation. This produces a smooth resolution-scaling curve rather than a piecewise one.
2. **Shift-equivariant spatial loss** (see [[shift-equivariant-distillation-loss_ranzinger2026]] — co-required): random patch-aligned crop-shifts for student and each teacher, loss computed only on shared positions via $\mathcal{F}_{S \to T}$. Student cannot mimic teachers' positional noise (register tokens, ViTDet window borders).
3. **Shift-equivariant MESA**: EMA-student self-matching with different crops between student and EMA, aligned via $\mathcal{F}_{S \to S}$. Layer-norm without affine.
4. **Angle-normalized (balanced) summary loss**: extends PHI-S dispersion balancing to summary tokens. Per-teacher angular dispersion (SigLIP2 0.694 vs DINOv3-7B 2.186, ~3× apart) normalizes the summary loss so no single teacher dominates the gradient:
   $$L_{\text{angle}}(x,y) = \frac{\Theta(x,y)^2}{\text{Disp}(\Theta_y)}$$

Deployment-side: **ViTDet mode** — student supports windowed attention (6×6 to 32×32 token windows, 4 global-attention layers interspersed) for drastic high-res speedup. Window size × patch size must divide input resolution.

Released as SO400M (412M) and H (631M) under NVIDIA Open Model License (commercial use permitted — first RADIO with this property).

## Trade-offs vs. the decomposed pipeline

Same structural trade as RADIOv2.5: capability frozen at distillation time. A 2027 SigLIP3 / DINOv4 / SAM4 triggers full retrain; Trident's three-backbone composition retains component-level upgradeability. Gain over RADIOv2.5 on this axis: **nothing** — C-RADIOv4 does not reduce the capability-freeze risk. The bet is still "frontier foundation models turn over slowly enough that a yearly RADIO retrain tracks them."

New trade surfaced by ViTDet mode: windowed attention on high-res images gives up some long-range modeling. The paper argues 4 global-attention layers are enough; only rigorous dense-prediction benchmarks will show whether the trade is benign.

## Why it wins

Ablations of individual refinements are **not published** — the paper reports full-system numbers only. Full-system evidence:

- ImageNet-1k zero-shot 82.51 → 83.09, kNN 85.81 → 86.59 (ViT-H vs RADIOv2.5-H).
- ADE20k mIoU 51.58 → **55.20** — matches DINOv3-7B (55.9) at 10× fewer parameters.
- VOC mIoU 85.97 → **87.24** — beats DINOv3-7B (86.6).
- Probe3D NAVI 60.89 → **63.44**, SPair 56.24 → **60.57** — C-RADIOv4-H is the best RADIO on keypoint/category correspondence.
- Resolution scaling holds up to 1536px (trained to 1152px): 57.72 mIoU vs DINOv3-7B's 57.8.
- Figure 1/2: visibly cleaner object boundaries and no register-token/ViTDet-border speckles in PCA visualization vs C-RADIOv3-H and DINOv3.
- ViTDet-8/12 modes: SO400M-VDT12 faster than SAM3's ViT-L+ at same setting.

Weakness: SA-Co/Gold cgF1 44.7 (best C-RADIOv4) vs 54.1 (SAM3) — 10-point gap on instance segmentation, largest on domain-specific queries (fg_sports_equipment, wiki_common). SAM3's vision encoder retains signal C-RADIOv4 doesn't capture on these domains.

## Preconditions & compatibility

- **License**: NVIDIA Open Model License (commercial-use permitted). This is the principal delta from RADIOv2.5's NSCL — unblocks every production integration previously gated on research-only use.
- **Teachers**: SigLIP2-g-384, DINOv3-7B, SAM3 must all be accessible at training time. The bet's payoff depends on teacher quality remaining the ceiling.
- **Co-required idea**: [[shift-equivariant-distillation-loss_ranzinger2026]] is the distinctive lever — not a removable piece. A C-RADIOv4-style recipe without shift-equivariance is essentially RADIOv2.5 with upgraded teachers.
- **Downstream**: consumers must use the single unified student backbone. A SAM3-replacement variant (`sam3-radio` fork) swaps the vision encoder of SAM3, keeping the decoder unchanged.

## Pipeline-shape implications

Collapses three stages into one student backbone (same as predecessor; `refines:` edge, not a scope change). Threads using op:distilled-single-pass represent this as a composite node spanning all three OV-2D stages.

Enables a **new specialization pattern**: SAM3-encoder drop-in. Treat C-RADIOv4 as a plug-compatible replacement for SAM3's ViT-L+ encoder (keeping SAM3's mask decoder). This is a bridge-like use (one backbone substituted into another model's pipeline) but does not require a first-class bridge idea — the adaptor machinery makes it just a configuration.

## Trade-offs vs. RADIOv2.5 (the specific refinement)

What you gain over the predecessor:
- Commercial license (big practical unblock).
- Better dense performance (ADE20k +3.6 mIoU; VOC +1.3 mIoU; NAVI/SPair +2.5/+4.3).
- Cleaner feature boundaries (positional-noise hygiene).
- Any-resolution support, not just the two training resolutions.
- ViTDet deployment mode for high-res latency.

What you **don't** gain:
- Prompt-conditioning capabilities (SAM3 has these; the distilled student does not carry them over — see Bet #018 in [[open-vocab-2d-composition]] for the residual open direction).
- Modularity vs. Trident — still frozen at distillation time.
- Ablation-level clarity on which training-side change matters most.

## Open questions

- Which of the four training-side refinements contributes most? Without per-change ablations, it's unclear if the gains are dominated by the teacher upgrade (SigLIP2/DINOv3/SAM3) or the loss/schedule changes. Downstream adopters targeting one subsystem (e.g., per-scene distillation) can't cherry-pick cleanly.
- Does angle-normalized summary loss hurt when teachers' angular distributions drift during training? Not tested.
- Can C-RADIOv4 be used as a drop-in in per-scene 3DGS feature distillation (LangSplat, Gaussian Grouping) despite its different feature dimension and adaptor structure vs CLIP? Open experimental question — captured as Bet #014 in [[lifting-foundation-models-to-3d]].
- What's the training budget relative to RADIOv2.5? Paper doesn't report. Plausibly higher due to stochastic resolutions and paired shifted-crop passes.
