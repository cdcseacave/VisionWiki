---
title: Foundation Model
type: concept
tags: [foundation-model, pretraining, transfer-learning, frozen-features]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/radford2021_clip.md, wiki/papers/oquab2023_dinov2.md, wiki/papers/kirillov2023_sam.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/carion2026_sam-3.md]
status: draft
---

## What it is

A **foundation model** is a large pretrained neural network — typically a [[vision-transformer|ViT]] or multimodal dual-encoder — that produces general-purpose representations transferable across a wide range of downstream tasks without per-task retraining. In vision, the canonical families as of 2024–2026 are:

| Family | Canonical model(s) | Pretraining signal | Primary output |
|---|---|---|---|
| Vision-only self-supervised | [[dinov2|DINOv2]] · [DINOv3](../papers/simeoni2025_dinov3.md) | [[self-supervised-learning]] (DINO + iBOT losses) | Dense patch features |
| Vision-language contrastive | [[clip|CLIP]] · SigLIP · OpenCLIP | Image–text contrastive | Image-level embeddings aligned with text |
| Promptable segmentation | [[sam|SAM]] · [SAM 3](../papers/carion2026_sam-3.md) | SA-1B / SA-Co data engines | Masks for point/box/mask/text prompts |
| Agglomerative distillation | [RADIOv2.5](../papers/heinrich2025_radiov25.md) | Multi-teacher distillation (CLIP + DINOv2 + SAM) | Unified backbone covering all three capabilities |

## The "frozen-backbone + task-head" pattern

2024–2026 geometric-CV is dominated by pipelines that **freeze a foundation backbone** and train only a lightweight task head on top. Examples in this wiki:

- [[dust3r|DUSt3R]] · [[mast3r|MASt3R]] · [[vggt|VGGT]] · [[jang2025_pow3r|Pow3R]] — frozen DINOv2 features feed a geometric regression head.
- [[edstedt2025_roma-v2|RoMa v2]] — frozen DINOv3 features drive dense correspondence.
- [[shi2024_open-vocab-segmentation|Trident]] — composes frozen CLIP + DINO + SAM for training-free open-vocab segmentation.
- [[qin2024_langsplat|LangSplat]] · [[jiao2025_clip-gs|CLIP-GS]] · [[ye2024_gaussian-grouping|Gaussian Grouping]] — distill/align frozen 2D foundation features into 3DGS.

See the [[foundation-features-for-geometry]] and [[lifting-foundation-models-to-3d]] threads for the synthesis.

## Why freeze?

- **Robustness**: frozen features are OOD-tested at web scale; fine-tuning on a small task set usually erodes generalization.
- **Efficiency**: train a task head in hours vs. backbone retraining in GPU-weeks.
- **Composability**: the same backbone supports many heads; swap task heads without retraining.
- **Reproducibility**: backbone release (weights + recipe) is a known artifact; task heads only need to reference it.

## Failure modes
- **Domain gap**: medical, microscopy, aerial, and thermal imagery often require fine-tuning or domain-specific pretraining (RADIO's aerial variants, BioMedCLIP).
- **Granularity mismatch**: CLIP is image-level; using it for dense prediction needs spatial lifting (DINO spatial affinity, SAM masks).
- **Token/patch resolution**: ViT backbones typically operate at 14–16px patches; ultra-fine structure (hair, thin wires) is lost without multi-scale tricks.

## Key references
- [Radford et al. 2021 (CLIP)](../papers/radford2021_clip.md)
- [Oquab et al. 2023 (DINOv2)](../papers/oquab2023_dinov2.md)
- [Kirillov et al. 2023 (SAM)](../papers/kirillov2023_sam.md)
- [Siméoni et al. 2025 (DINOv3)](../papers/simeoni2025_dinov3.md)
- [Heinrich et al. 2025 (RADIOv2.5)](../papers/heinrich2025_radiov25.md)
- [Carion et al. 2026 (SAM 3)](../papers/carion2026_sam-3.md)
