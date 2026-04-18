---
title: Open-Vocabulary 2D Composition (CLIP + DINO + SAM)
type: thread
tags: [open-vocabulary, segmentation, foundation-model, training-free, composition]
created: 2026-04-15
updated: 2026-04-18
sources: [wiki/papers/shi2024_open-vocab-segmentation.md, wiki/papers/carion2026_sam-3.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/radford2021_clip.md, wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/kirillov2023_sam.md, wiki/papers/shi2026_self-distilled-roi.md]
operating_points: [op:modular-training-free, op:unified-promptable, op:distilled-single-pass]
status: draft
---

## Goal

Segment or label arbitrary concepts in a 2D image without per-class training, using frozen foundation models. Better = higher open-vocabulary mIoU / AP at equal or lower inference cost, robust across closed-vocab → open-vocab shift, and **modular** enough to absorb a new frontier backbone without retraining.

## Goal contract (optional, structured)

```yaml
metric: [open-vocab-mIoU, per-image-inference-ms, cross-vocab-robustness]
target_regime: [static-image | video, any-prompt-type text | exemplar | click]
constraints: [zero-per-class-training, frozen-backbones-only-for-training-free-lane, commercial-license-ok]
required_capabilities: [semantic-alignment, spatial-coherence, mask-boundary-precision]
```

## SOTA pipelines

### op:modular-training-free (current default when modularity matters; cost = 3 ViT passes)

1. **Semantic alignment**: frozen CLIP (OpenCLIP ViT-L/14 @ 336 is the de-facto choice) produces text-aligned per-region similarity scores from text prompts. Paper: [[radford2021_clip]]. Failure mode: image-level training → weak on spatial precision.
2. **Spatial coherence / affinity**: frozen DINO self-similarity propagates the semantic signal to spatially coherent regions. Paper: [[oquab2023_dinov2]] → [[simeoni2025_dinov3]] (DINOv3 is the upgraded default). Gain: DINOv3 Gram anchoring gives cleaner patch affinities at high res.
3. **Mask quality**: frozen SAM snaps the aggregated signal to clean mask boundaries. Paper: [[kirillov2023_sam]].
4. **Aggregation**: Trident's "Feature Splicing → Spatial Correlation → Global Aggregation". Paper: [[shi2024_open-vocab-segmentation]]. Cost: three ViT forward passes per image.

### op:unified-promptable (current default when inference latency matters and video support is needed)

1. **Single SAM 3 forward pass** takes text prompts (noun phrases) or image exemplars and emits masks + instance IDs + video tracks. Paper: [[carion2026_sam-3]]. Trained on SA-Co (4M concept labels). Collapses CLIP + SAM into one backbone with tracking built in.

### op:distilled-single-pass (current default when inference budget is tightest)

1. **Single RADIOv2.5 forward pass** — outputs subsume DFN-CLIP + SigLIP + DINOv2 + SAM teacher capabilities. Paper: [[heinrich2025_radiov25]]. Task head on the unified features. Trade: capability frozen at distillation time.

## Pipeline lineage

- 2021 · op:modular-training-free · semantic alignment: task-specific classifiers → CLIP zero-shot · driver: [[radford2021_clip]].
- 2023 · op:modular-training-free · mask quality: Mask R-CNN class-specific masks → SAM prompt-conditioned masks · driver: [[kirillov2023_sam]].
- 2023 · op:modular-training-free · spatial coherence: hand-crafted affinity / GrabCut → DINOv2 self-similarity · driver: [[oquab2023_dinov2]].
- 2024 · op:modular-training-free · aggregation: per-paper bespoke fusion → Trident three-stage pipeline · driver: [[shi2024_open-vocab-segmentation]].
- 2025 · op:modular-training-free · spatial coherence: DINOv2 → DINOv3 · driver: [[simeoni2025_dinov3]].
- 2025 · op:distilled-single-pass · new OP: three-ViT inference → single distilled backbone · driver: [[heinrich2025_radiov25]].
- 2026 · op:unified-promptable · new OP: CLIP + SAM separate → SAM 3 single model · driver: [[carion2026_sam-3]].
- 2026 · op:modular-training-free · zero-label RoI selection: external proposer → MLLM self-distilled attention · driver: [[shi2026_self-distilled-roi]] (adjacent pattern; not yet in the main pipelines).

## Candidate components / not yet integrated

- **SigLIP** (Zhai 2023, no wiki page yet) — pairwise sigmoid contrastive. Drop-in candidate for CLIP in Pipeline A's semantic-alignment stage. Expected gain: ~2–4% open-vocab mIoU at equal compute, larger effective batch. Blocked on: no stub; no direct Pipeline-A ablation comparing CLIP vs SigLIP in the Trident composition.
- **SAM 2** (Ravi 2024, no wiki page) — video SAM. Candidate for Pipeline A's mask stage when video is needed, before committing to full SAM 3. Blocked on: stub.
- **MLLM self-attention as spatial signal** (SD-RPN pattern, [[shi2026_self-distilled-roi]]) — could replace DINO in the spatial-coherence stage of Pipeline A. Blocked on: MLLM dependency; per-backbone threshold tuning.

## Open questions & synthesis bets

### Bet #017 — Trident with SigLIP + DINOv3 + SAM 3
status: proposed
combines: [[carion2026_sam-3]], [[simeoni2025_dinov3]], [[shi2024_open-vocab-segmentation]]
stage_target: open-vocab-2d.composition
op_target: op:modular-training-free
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: Keep Trident's three-stage structure but upgrade each leg to the 2026 state: SigLIP (semantic) + DINOv3 (spatial) + SAM 3 (mask + instance). Trident's modularity + SAM 3's instance tracking, at the cost of three-backbone inference.
expected_gain: 2–4% open-vocab mIoU over Trident-original + video / instance tracking for free.
risk: SAM 3 already does text prompts; using Trident's aggregation on top may be redundant. Three-backbone inference cost may push users to Pipeline C.
validating_experiment: Ablate {original Trident, +SigLIP, +DINOv3, +SAM3, full upgrade} on ADE20K open-vocab + video-object-segmentation benchmarks. Needs SigLIP stub page first.
triggers: [ingest-of-idea:siglip-wiki-page, benchmark:video-trident]
created: 2026-04-15 · updated: 2026-04-18

### Bet #018 — Distill SAM 3 + DINOv3 into a RADIO-style student
status: proposed
combines: [[heinrich2025_radiov25]], [[carion2026_sam-3]], [[simeoni2025_dinov3]]
stage_target: open-vocab-2d.unified-backbone
op_target: op:distilled-single-pass
confidence: med
magnitude: substantial
cost: months
breakage_risk: med
hypothesis: RADIO distillation recipe applied to SAM 3 + DINOv3 (instead of RADIOv2.5's original teacher set) gives Pipeline B's promptable-concept capabilities at Pipeline C's single-pass inference cost.
expected_gain: Pipeline B capabilities (video, instance IDs, text prompts) in a single ViT forward pass; ~3× inference speedup over running SAM 3 + DINOv3 separately.
risk: SAM 3's prompt-conditioning head doesn't distill into a classifier-free student trivially. RADIO recipe may not generalize to prompt-based teachers.
validating_experiment: Train RADIO-style agglomerative student on SAM 3 + DINOv3; compare vs. Pipeline B (SAM 3 alone) on segmentation + vs. Pipeline C (RADIOv2.5) on spatial benchmarks.
triggers: [ingest-of-idea:prompt-conditional-distillation]
created: 2026-04-15 · updated: 2026-04-18

### Bet #019 — SD-RPN attention-denoising on DINOv3 as spatial stage
status: proposed
combines: [[shi2026_self-distilled-roi]], [[simeoni2025_dinov3]]
stage_target: open-vocab-2d.spatial-coherence
op_target: op:modular-training-free
confidence: med
magnitude: incremental
cost: weeks
breakage_risk: low
hypothesis: SD-RPN's attention-denoising (remove sink tokens, threshold into foreground/background) was developed for MLLM cross-attention but the recipe is general. Applying it to DINOv3 self-attention may produce cleaner spatial-coherence signals for Trident.
expected_gain: 1–3% mIoU improvement on fine-structure queries; cleaner object boundaries in Trident output.
risk: Sink-token-removal thresholds are per-backbone; DINOv3 may need different parameters than MLLM attention. Marginal gain over existing DINO self-similarity.
validating_experiment: Replace DINOv3 self-similarity with SD-RPN-denoised attention in Trident; ablate threshold choices on ADE20K open-vocab.
triggers: [ingest-of-idea:cross-backbone-attention-denoising]
created: 2026-04-15 · updated: 2026-04-18

## Capability gaps

- **Compositional-prompt reasoning** ("red chair *without* armrests") — CLIP is weak; SAM 3 exemplars partly help but don't generalize. Would unlock: compositional-query benchmarks. Search target: VLM + SAM 3 hybrids with explicit logical reasoning.
- **Fine-structure segmentation at sub-patch scale** — all three backbones operate at 14–16 px patches. Would unlock: thin-object benchmarks. Search target: dual-resolution or cascaded-ViT approaches.
- **Harmonized open-vocab benchmark** — cross-paper numbers are apples-to-oranges. Not a paper-search target; a meta-benchmark gap the thread must track.

## Contradictions & tensions

- Cross-paper benchmarks in this space are frequently apples-to-oranges (vocabulary size, class balance, evaluation subtask). Pipeline A/B/C headline numbers are not directly comparable without a harmonized benchmark; the thread's SOTA-pipeline claims are *strategy-conditional*, not absolute.

## Working hypothesis

The 2D foundation models dominant in 2023–2026 — [[clip|CLIP]] (semantics), [[dinov2|DINO]] (spatial coherence), [[sam|SAM]] (mask boundaries) — are **complementary**, and the most effective open-vocabulary 2D pipelines are those that **compose** them cleanly, either at inference time (training-free) or by distilling them into one backbone. The composition pattern transfers directly to 3D lifting methods (see [[lifting-foundation-models-to-3d]]).

## Composition strategies

### Training-free inference-time composition (Trident pattern)
- [[shi2024_open-vocab-segmentation|Trident (Shi et al. 2024)]] — "Feature Splicing → Spatial Correlation → Global Aggregation":
  1. **CLIP** supplies per-region semantic alignment with the text prompt.
  2. **DINO** affinity / self-similarity propagates the semantic signal to spatially coherent regions.
  3. **SAM** supplies clean mask boundaries; the aggregated semantic score is snapped to SAM masks.
- Advantage: zero training, modular, swap components as backbones improve.
- Cost: runtime = sum of three ViT inferences per image.

### Unified promptable-concept model (SAM 3 pattern)
- [[carion2026_sam-3|SAM 3]] — single model trained on SA-Co (4M concept labels). Takes text prompts (noun phrases) or image exemplars directly; outputs masks + instance IDs + video tracking. Collapses CLIP-style text alignment and SAM-style mask quality into one backbone.
- Advantage: single-pass inference, video support.
- Cost: requires a specialized data engine; less modular if one component improves.

### Multi-teacher distillation (RADIO pattern)
- [[heinrich2025_radiov25|RADIOv2.5]] — agglomerative distillation from DFN-CLIP + SigLIP + DINOv2 + SAM into one encoder. The result is a single backbone whose features subsume the capabilities of each teacher — training-time composition as an alternative to inference-time composition.
- Advantage: one forward pass replaces three at inference.
- Cost: requires access to all teachers; capability gets locked at distillation time.

## Why it works

CLIP and DINO are on **opposite corners of a capability diagram**:

|            | Semantic alignment | Spatial coherence |
|------------|-------------------|-------------------|
| [[clip\|CLIP]]   | **Strong** (text-aligned) | Weak (image-level training) |
| [[dinov2\|DINO]] | Weak (no text) | **Strong** (patch-level SSL) |
| [[sam\|SAM]]    | None (concept-agnostic) | Precise boundaries |

None is individually sufficient for open-vocabulary dense prediction. Composition covers the deficit.

## Where it breaks

- **Compositional prompts** ("the red chair *without* armrests") — CLIP's image-text alignment is weak on composition; neither DINO nor SAM adds logical reasoning. SAM 3's exemplar prompts partly compensate.
- **Small / thin objects** — all three backbones operate at 14–16 px patches; fine structure is lost.
- **Evaluation** — benchmarks vary on vocabulary size, class balance, and which subtask is emphasized. Cross-paper numbers are frequently apples-to-oranges.

## Relation to 3D

Every 3D-lifting method in [[lifting-foundation-models-to-3d]] *inherits* this composition pattern. LangSplat is "LERF with the SAM piece fixed." Gaussian Grouping is "SAM lifted." CLIP-GS is "CLIP-aligned 3DGS encoder, Trident's CLIP role but over whole scenes." Understanding the 2D composition is load-bearing for reading 3D segmentation papers.

## Open questions
- Does the composition pattern survive as each component improves independently, or does one backbone eventually subsume the others?
- SAM 3 unifies CLIP + SAM capabilities — does a future "DINO 3" or "SigLIP 3" subsume the DINO role too, collapsing the Trident pattern into one model?
- At what composition cost does inference latency push users toward distilled models (RADIO)?

## Adjacent patterns: self-distilled attention as a zero-label signal

- [SD-RPN (Shi 2026)](../papers/shi2026_self-distilled-roi.md) is not a
  CLIP+DINO+SAM composition paper but lands in the same neighborhood: it
  takes an MLLM's **own internal cross-attention**, denoises it by removing
  sink tokens, and uses the cleaned signal as a pseudo-label to train a
  Region Proposal Network — no external annotations. **Strengths**: +10%
  absolute accuracy on TextVQA / DocVQA / V-Star with only 10K QA pairs,
  single partial forward pass for RoI selection. **Weaknesses**: processes
  RoIs independently (no spatial relation between RoIs), depends on base
  MLLM attention quality, fixed thresholds ($\tau_{fg}$, $\tau_{bg}$,
  $\tau_{norm}$) may need per-backbone tuning.

The adjacency to this thread: the MLLM's attention is playing the same
*spatial-coherence* role that DINO self-similarity plays in Trident, just
sourced from a different frozen model (the MLLM itself). If the composition
hypothesis generalizes, every powerful frozen backbone exposes a usable
"where to look" signal that can be distilled into a task head with minimal
supervision — Trident uses DINO's, SD-RPN uses an MLLM's, and RADIOv2.5
distills several into one.
