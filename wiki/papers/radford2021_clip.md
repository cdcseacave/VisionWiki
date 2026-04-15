---
title: "Learning Transferable Visual Models From Natural Language Supervision (CLIP)"
type: paper
tags: [vision-language, contrastive-learning, foundation-model, zero-shot]
created: 2026-04-14
updated: 2026-04-15
sources: []
local_paper: papers/fundamentals/radford_2021_clip.pdf
url: https://arxiv.org/abs/2103.00020
status: stable
---

📄 [Full paper](../../papers/fundamentals/radford_2021_clip.pdf) · [arXiv](https://arxiv.org/abs/2103.00020) · [Project](https://openai.com/research/clip)

## TL;DR

Radford et al. (OpenAI, ICML 2021) train a **contrastive image-text dual encoder** on 400M image-caption pairs scraped from the web. The resulting model performs **zero-shot classification** on 30+ datasets at accuracy competitive with fully-supervised ResNet-50 — without ever seeing labeled training examples from those datasets.

## Problem

Supervised ImageNet pretraining hit diminishing returns and transferred poorly to out-of-distribution tasks. Natural-language supervision was known to be a richer signal (Learning Visual Features from Captions, DeVISE) but prior attempts had been compute-limited and produced weak models.

## Method

- **Dual encoder**: separate image encoder (ViT or ResNet) and text encoder (Transformer), each projecting to a shared embedding space.
- **Contrastive objective (InfoNCE)**: in a batch of N (image, caption) pairs, maximize cosine similarity of the N matching pairs vs. N²−N non-matching pairs. Symmetric cross-entropy over rows and columns of the similarity matrix.
- **Dataset (WIT)**: 400M image-text pairs scraped from the internet, filtered by query diversity.
- **Zero-shot inference**: build class-name embeddings via prompt templates ("a photo of a {class}"), classify by cosine similarity to the image embedding.

## Results

- Zero-shot ImageNet top-1 ≈ 76% (ViT-L/14 @ 336) — matches supervised ResNet-50.
- Strong generalization to out-of-distribution benchmarks (ImageNet-A/R/Sketch) where supervised models collapse.
- Consistent scaling: zero-shot accuracy grows predictably with compute.

## Why it matters

CLIP is the **pre-eminent vision-language embedding** and the workhorse of open-vocabulary 3D understanding: distilled into radiance fields (LERF), 3DGS (LangSplat, Feature-3DGS), and used as a semantic prior everywhere. Also the text-conditioning for diffusion models (Stable Diffusion, DALL-E 2).

## Pipeline contribution

- **Contrastive dual-encoder (N1)** — symmetric InfoNCE over N×N image-text similarity matrix with learned temperature. candidate thread: [[open-vocab-2d-composition]] · stage: *semantic alignment (text ↔ region)* · replaces/augments: *pre-2021 text-free classifiers* · expected gain: zero-shot mIoU transfer across 30+ datasets; the semantic-alignment leg of Trident (+4.2% over prior SOTA, Shi 2024).
- **Zero-shot prompt-embedding classifier (N2)** — class-name text embeddings enable arbitrary-vocabulary classification at inference. candidate thread: [[lifting-foundation-models-to-3d]] · stage: *2D semantic prior lifted to 3D* · replaces/augments: *per-scene object labels* · expected gain: open-vocab queries on 3DGS scenes (LangSplat distills CLIP per-Gaussian).
- **Web-scale contrastive data recipe (N3)** — 400M WIT pairs + predictable scaling law. Not a pipeline component but thread-level evidence that *frozen backbone + small head* will keep winning; supports the project-wide bet.
- **Not a component of** [[foundation-features-for-geometry]]: CLIP is image-level and text-aligned but spatially weak, so it is excluded from the geometry pipeline. SigLIP (not yet in the wiki) is the natural drop-in successor for the semantic-alignment lane.

## Relation to prior work

- Generalizes ConVIRT (medical image–text contrastive) to web-scale.
- Contrasted with [[dinov2|DINOv2]]: CLIP has language alignment; DINOv2 has stronger dense features.
- Successor/variant models: OpenCLIP (open replication), SigLIP (pairwise sigmoid loss), EVA-CLIP (masked image modeling pretraining).

## Open questions / limitations

- Zero-shot accuracy plateaus on fine-grained and compositional tasks.
- Embeddings entangle semantics with text-specific biases (prompt sensitivity).
- Dense prediction (segmentation, detection) requires additional adaptation — CLIP features are image-level.

## References added to the wiki

- [[clip]] (stub expanded).
