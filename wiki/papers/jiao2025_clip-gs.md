---
title: "CLIP-GS: Unifying Vision-Language Representation with 3D Gaussian Splatting"
type: paper
tags: [3dgs, multimodal, clip, representation-learning, retrieval]
created: 2026-04-15
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/jiao_2025_clip-gs.pdf
url: https://arxiv.org/abs/2412.19142
license_paper: CC-BY-4.0
status: stable
---

📄 [Full paper](../../papers/radiance-fields/jiao_2025_clip-gs.pdf) · [Supplement](../../papers/radiance-fields/jiao_2025_clip-gs_supplemental.pdf) · [arXiv](https://arxiv.org/abs/2412.19142) · _no code found (2026-04-15)_

## TL;DR

Jiao et al. (ICCV 2025) propose **CLIP-GS**, a multimodal representation that maps **[[3d-gaussian-splatting|3DGS]] ↔ image ↔ text** into a single [[clip|CLIP]]-aligned embedding space. Introduces a **GS Tokenizer** that converts 3DGS into serialized tokens processed by transformer layers initialized from point-cloud models, then contrasts against CLIP image/text embeddings. Outperforms point-cloud multimodal models (ULIP, Uni3D) across retrieval, zero-shot, and few-shot 3D tasks.

## Problem

3D multimodal models (ULIP, OpenShape, Uni3D) aligned **point clouds** with CLIP. But point clouds are sparse, texture-less, and a poor proxy for photorealistic 3D. 3DGS carries texture and geometry faithfully but had no native multimodal embedding. The research gap: how to align **3DGS** with vision-language foundation models.

## Method

- **GS Tokenizer**: Farthest-Point-Sampling + k-NN groups form "Gaussian patches"; each is serialized into a fixed-length token stream capturing position, color, scale, rotation, and opacity.
- **Transformer backbone** initialized with weights from point-cloud foundation models (Uni3D / OpenShape) — warm-start saves training compute and transfers geometric priors.
- **Contrastive training**: triplets of (3DGS scene, rendered image, text caption). Symmetric InfoNCE aligns the 3DGS embedding with CLIP's visual and textual embeddings.
- **Image voting loss**: rendered multi-view images "vote" on the gradient direction during optimization — encourages the 3DGS embedding to be rotation/pose-invariant.
- **Triplet generation pipeline**: efficient synthesis of (3DGS, image, text) triplets from existing 3D datasets + rendered views.

## Results

- Outperforms point-cloud-based ULIP / Uni3D / OpenShape on: 3D multimodal retrieval, zero-shot classification, few-shot classification.
- CLIP-GS-B has ~9M parameters and trains in ~14 h on 8×A6000.
- Scales predictably with model size and training triplets.

## Why it matters

Establishes **3DGS as a first-class citizen in 3D foundation-model learning**, displacing point clouds as the canonical multimodal 3D input. Enables downstream zero-shot 3D tasks without training a scene-specific language field (contrast with [[qin2024_langsplat|LangSplat]]'s per-scene autoencoder).

## Pipeline contribution

- **GS Tokenizer: 3DGS → token stream via FPS + k-NN patches (N1)** — serializes Gaussian patches capturing position, color, scale, rotation, opacity. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline III · stage: *3DGS → embedding* · replaces/augments: *point-cloud tokenizers (ULIP, Uni3D)* · expected gain: 3DGS as a first-class multimodal 3D input; SOTA retrieval + zero-shot + few-shot over point-cloud methods.
- **Transformer init from point-cloud FM (Uni3D/OpenShape) (N2)** — warm-starts geometric priors for the 3DGS encoder. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline III · stage: *transfer learning* · expected gain: cheaper training; leverages decades of point-cloud pretraining.
- **Scene-level contrastive with CLIP image+text (N3)** — triplets (3DGS, rendered images, text); symmetric InfoNCE. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline III · stage: *alignment objective* · replaces/augments: *per-scene distillation (LangSplat)* · expected gain: zero-shot cross-scene queries.
- **Image-voting loss (N4)** — multi-view rendered images vote on gradient direction; rotation/pose invariance. candidate thread: *contrastive 3D training* · stage: *invariance regularization* · expected gain: pose-invariant embeddings.
- **Role**: CLIP-GS is the **generalizable lane** of the thread — opposite axis-2 choice from LangSplat's per-scene distillation. The open tension "per-scene quality vs. zero-shot transfer" lives between these two papers.

## Relation to prior work

- Sibling philosophy: [[qin2024_langsplat|LangSplat]] distills CLIP **into** a per-scene 3DGS field (scene-specific); CLIP-GS aligns 3DGS **with** CLIP at the scene/object level (generalizable).
- Succeeds ULIP, Uni3D, OpenShape (point-cloud-based).
- Builds on [[radford2021_clip|CLIP]] as the anchor modality.

## Open questions / limitations

- 3DGS must be pre-computed (not reconstructed jointly with the multimodal alignment).
- GS Tokenizer's serialization order is heuristic; performance sensitivity to ordering not fully characterized.
- Scene-scale tokenization (thousands of Gaussians) still strained; works best on object-scale captures.

## Code & license

No code released for Jiao et al. 2412.19142 as of 2026-04-15. Name-collision warning: the repo `github.com/gbliao/CLIP-GS` belongs to a *different* paper (Liao et al. 2404.14249) — do not link it here.

## References added to the wiki

- [[3d-gaussian-splatting]], [[clip]] — cross-links + lineage.
