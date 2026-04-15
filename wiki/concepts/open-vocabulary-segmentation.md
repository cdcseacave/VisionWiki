---
title: Open-Vocabulary Segmentation
type: concept
tags: [open-vocabulary, segmentation, clip, sam, dino, training-free]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/shi2024_open-vocab-segmentation.md, wiki/papers/carion2026_sam-3.md, wiki/papers/qin2024_langsplat.md]
status: draft
---

## What it is

**Open-vocabulary segmentation (OVS)** is the task of producing segmentation masks for arbitrary concepts specified by **free-form text at inference time**, without having seen those classes during training. It sits at the top of the [[segmentation|segmentation task taxonomy]] — generalizing over the fixed-vocabulary semantic/instance/panoptic setups and over the mask-only promptable setting.

## The Trident composition template

[[shi2024_open-vocab-segmentation|Trident (Shi et al. 2024)]] crystallized a **training-free** OVS recipe composing three foundation models:

| Role | Provider | Signal |
|---|---|---|
| Semantic alignment (text ↔ region) | [[clip|CLIP]] | Cosine similarity between text prompt and image-region embeddings |
| Spatial coherence / affinity | [[dinov2|DINOv2/DINOv3]] | Self-similarity of patch features → within-object smoothness |
| Mask boundaries | [[sam|SAM]] | Precise per-mask outlines |

**Composition flow**: extract CLIP region scores → propagate through a DINO affinity graph → snap to SAM masks for clean boundaries. No training.

This pattern has become the **reference baseline** for training-free OVS and has been absorbed (implicitly) into promptable-concept-segmentation systems.

## The promptable-concept variant

[[carion2026_sam-3|SAM 3]] collapses OVS and promptable segmentation into a single task — **Promptable Concept Segmentation (PCS)**: given a noun phrase, image exemplars, or both, return masks *and unique instance IDs* for all matching objects in image or video. Achieves the OVS goal via a single trained model rather than foundation-model composition, and adds video identity tracking.

- **Trident-style composition**: zero training, modular, easy to swap components as backbones improve. Cost: multi-model inference latency.
- **SAM-3-style unified model**: faster at inference, supports video, but requires specialized data (SA-Co, 4M concept labels).

## Lifting OVS to 3D

Open-vocabulary *3D* segmentation (query "the red sofa" in a reconstructed scene) inherits the 2D recipe:

- **Per-Gaussian CLIP features** — [[qin2024_langsplat|LangSplat]] distills CLIP hierarchy-aligned with SAM masks into 3DGS, enabling text-query → rendered 3D mask. 199× faster than the NeRF-based LERF predecessor.
- **Scene-level CLIP alignment** — [[jiao2025_clip-gs|CLIP-GS]] aligns a 3DGS scene embedding with CLIP text, supporting retrieval and zero-shot classification.
- **Identity-only predecessor** — [[ye2024_gaussian-grouping|Gaussian Grouping]] handles promptable 3D segmentation (SAM masks lifted to per-Gaussian identity) but lacks text alignment — OVS requires combining with a CLIP/language step.

See [[lifting-foundation-models-to-3d]] for the full synthesis.

## Open questions
- Fine-grained compositional prompts ("red chair *without* armrests") — still weak.
- Evaluation protocol for open-vocab: class-agnostic mask quality vs. text-alignment precision vs. recall at different vocabulary sizes.
- 3D OVS benchmarks are nascent; most papers report on LERF's small handful of scenes.

## Key references
- [Shi et al. 2024 (Trident)](../papers/shi2024_open-vocab-segmentation.md) — training-free composition.
- [Carion et al. 2026 (SAM 3)](../papers/carion2026_sam-3.md) — unified PCS model.
- [Qin et al. 2024 (LangSplat)](../papers/qin2024_langsplat.md) — 3D-lifted OVS on 3DGS.
