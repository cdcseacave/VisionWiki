---
title: Open-Vocabulary 2D Composition (CLIP + DINO + SAM)
type: thread
tags: [open-vocabulary, segmentation, foundation-model, training-free, composition]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/shi2024_open-vocab-segmentation.md, wiki/papers/carion2026_sam-3.md, wiki/papers/heinrich2025_radiov25.md]
status: draft
---

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
