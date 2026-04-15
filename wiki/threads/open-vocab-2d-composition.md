---
title: Open-Vocabulary 2D Composition (CLIP + DINO + SAM)
type: thread
tags: [open-vocabulary, segmentation, foundation-model, training-free, composition]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/shi2024_open-vocab-segmentation.md, wiki/papers/carion2026_sam-3.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/radford2021_clip.md, wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/kirillov2023_sam.md, wiki/papers/shi2026_self-distilled-roi.md]
status: draft
---

## Goal & success criteria

Segment or label arbitrary concepts in a 2D image without per-class training, using frozen foundation models. "Better" = higher open-vocabulary mIoU / AP at equal or lower inference cost, robust across closed-vocab → open-vocab shift, and **modular** enough to absorb a new frontier backbone without retraining.

## Current SOTA pipeline (as of 2026-04-15)

Three *parallel* pipelines are viable — the thread hasn't collapsed to one:

**Pipeline A — Training-free Trident composition** (current default when modularity matters):
1. **Semantic alignment**: frozen CLIP (OpenCLIP ViT-L/14 @ 336 is the de-facto choice) produces text-aligned per-region similarity scores from text prompts. Paper: [radford2021_clip]. Failure mode: image-level training → weak on spatial precision.
2. **Spatial coherence / affinity**: frozen DINO self-similarity propagates the semantic signal to spatially coherent regions. Paper: [oquab2023_dinov2] → [simeoni2025_dinov3] (DINOv3 is the upgraded default). Gain: DINOv3 Gram anchoring gives cleaner patch affinities at high res.
3. **Mask quality**: frozen SAM snaps the aggregated signal to clean mask boundaries. Paper: [kirillov2023_sam].
4. **Aggregation**: Trident's "Feature Splicing → Spatial Correlation → Global Aggregation". Paper: [shi2024_open-vocab-segmentation]. Cost: three ViT forward passes per image.

**Pipeline B — Unified promptable-concept model** (current default when inference latency matters and video support is needed):
1. **Single SAM 3 forward pass** takes text prompts (noun phrases) or image exemplars and emits masks + instance IDs + video tracks. Paper: [carion2026_sam-3]. Trained on SA-Co (4M concept labels). Collapses CLIP + SAM into one backbone with tracking built in.

**Pipeline C — Distilled unified backbone** (current default when inference budget is tightest):
1. **Single RADIOv2.5 forward pass** — outputs subsume DFN-CLIP + SigLIP + DINOv2 + SAM teacher capabilities. Paper: [heinrich2025_radiov25]. Task head on the unified features. Trade: capability frozen at distillation time.

## Pipeline lineage

- 2021 · semantic alignment: task-specific classifiers → CLIP zero-shot · driver: [radford2021_clip].
- 2023 · mask quality: Mask R-CNN class-specific masks → SAM prompt-conditioned masks · driver: [kirillov2023_sam].
- 2023 · spatial coherence: hand-crafted affinity / GrabCut → DINOv2 self-similarity · driver: [oquab2023_dinov2].
- 2024 · aggregation: per-paper bespoke fusion → Trident three-stage pipeline · driver: [shi2024_open-vocab-segmentation].
- 2025 · spatial coherence: DINOv2 → DINOv3 · driver: [simeoni2025_dinov3].
- 2025 · compute profile: three-ViT inference → single distilled backbone · driver: [heinrich2025_radiov25] (Pipeline C split off).
- 2026 · unified promptable segmentation: CLIP + SAM separate → SAM 3 single model · driver: [carion2026_sam-3] (Pipeline B split off).
- 2026 · zero-label RoI selection: external proposer → MLLM self-distilled attention · driver: [shi2026_self-distilled-roi] (adjacent pattern; not yet in the main pipelines).

## Candidate components / not yet integrated

- **SigLIP** (Zhai 2023, no wiki page yet) — pairwise sigmoid contrastive. Drop-in candidate for CLIP in Pipeline A's semantic-alignment stage. Expected gain: ~2–4% open-vocab mIoU at equal compute, larger effective batch. Blocked on: no stub; no direct Pipeline-A ablation comparing CLIP vs SigLIP in the Trident composition.
- **SAM 2** (Ravi 2024, no wiki page) — video SAM. Candidate for Pipeline A's mask stage when video is needed, before committing to full SAM 3. Blocked on: stub.
- **MLLM self-attention as spatial signal** (SD-RPN pattern, [shi2026_self-distilled-roi]) — could replace DINO in the spatial-coherence stage of Pipeline A. Blocked on: MLLM dependency; per-backbone threshold tuning.

## Open questions & synthesis bets

- Does the composition pattern survive as each component improves independently? Trident survived CLIP→SigLIP and DINOv2→DINOv3. **Open**: does it survive CLIP+SAM→SAM 3? If SAM 3 fully subsumes two of three legs, Pipeline A is structurally dominated.
- **Synthesis bet 1**: *Pipeline A with SigLIP-swapped semantic stage + DINOv3 spatial stage + SAM 3 mask stage.* Mixes [carion2026_sam-3] (mask) with [simeoni2025_dinov3] (spatial) and SigLIP (semantic). No paper does this. Expected gain: Trident's modularity + SAM 3's instance tracking, at the cost of three-backbone inference; usable for video datasets where SAM 3's video support is the primary draw but DINOv3's spatial precision still matters at per-frame scale.
- **Synthesis bet 2**: *Distill SAM 3 + DINOv3 into a RADIO-style agglomerative student* to get Pipeline B's capabilities at Pipeline C's inference cost. Mixes [heinrich2025_radiov25] + [carion2026_sam-3] + [simeoni2025_dinov3]. Not proposed by any paper.
- **Synthesis bet 3**: *SD-RPN attention-denoising applied to the DINO stage* — take DINOv3's attention, remove sink tokens per the SD-RPN recipe, use as the spatial-coherence signal. Mixes [shi2026_self-distilled-roi] + [simeoni2025_dinov3]. Not tried.

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
