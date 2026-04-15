---
title: Self-Supervised Learning (vision)
type: concept
tags: [self-supervised, pretraining, contrastive, distillation]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md]
status: draft
---

## What it is

**Self-supervised learning (SSL)** in vision trains representations on large image datasets without human labels, by constructing a supervisory signal from the images themselves. In the 2021–2026 era the dominant recipes are:

| Recipe | Canonical models | Idea |
|---|---|---|
| **Contrastive / distillation** (DINO family) | DINO · [[dinov2|DINOv2]] · [DINOv3](../papers/simeoni2025_dinov3.md) | Student/teacher [[vision-transformer|ViT]]: two augmented crops of one image should embed similarly; centering + sharpening prevent collapse. |
| **Masked image modeling (MIM)** | MAE · iBOT · BEiT | Predict masked patch contents from visible ones — encourages learning spatial structure. |
| **Hybrid** | DINOv2 / v3 | DINO image-level loss + iBOT patch-level loss + KoLeo regularizer. |

Contrast with **[[clip|CLIP]]-style vision-language training**: that is technically "self-supervised" (no manual labels) but relies on natural-language captions as a proxy signal. In this wiki "SSL" usually means the vision-only variant.

## Why it matters

SSL produces the strongest frozen **dense** features in 2025–2026 — better than supervised ImageNet pretraining on segmentation/depth/correspondence, and better than CLIP at anything requiring spatial coherence. DINOv2/v3 features are the default frozen backbone for 2024–2026 feed-forward 3D pipelines ([[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]], [[edstedt2025_roma-v2|RoMa v2]]).

## The scaling story

- **Data**: curated 142M image dataset (LVD-142M) for DINOv2 → larger curated + aerial augmentation for DINOv3.
- **Model**: ViT-g/14 for DINOv2 → distilled smaller variants (S/B/L) preserve quality. DINOv3 extends both axes.
- **Training dynamics**: DINOv3 discovered that **dense features degrade** during long training runs even as image-level features keep improving. Fixed via **Gram anchoring** — regularize the patch-feature Gram matrix toward a reference snapshot. See [[[simeoni2025_dinov3]]].
- **Post-hoc adaptations**: variable resolution, text-encoder retrofit (CLIP-style alignment on frozen backbone) — the "foundation model" becomes more flexible without retraining.

## Relation to other pretraining paradigms
- **Supervised ImageNet pretraining** — displaced at scale; SSL wins on dense tasks and OOD robustness.
- **Vision-language** ([[clip|CLIP]]) — complementary. CLIP is text-aligned, DINO is spatially coherent. RADIOv2.5 merges them via multi-teacher distillation; DINOv3's post-hoc text alignment hybridizes from the SSL side.

## Open questions
- Does scaling law keep going? Both data curation and model size have not obviously saturated.
- How transferable are SSL features to non-natural domains (medical, scientific)?
- What is the principled reason Gram anchoring works? Empirical fix without full theory.

## Key references
- [Oquab et al. 2023 (DINOv2)](../papers/oquab2023_dinov2.md)
- [Siméoni et al. 2025 (DINOv3)](../papers/simeoni2025_dinov3.md)
- [Heinrich et al. 2025 (RADIOv2.5)](../papers/heinrich2025_radiov25.md) — multi-teacher distillation combining SSL + vision-language.
