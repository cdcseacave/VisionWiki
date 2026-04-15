---
title: "Segment Anything"
type: paper
tags: [segmentation, foundation-model, promptable, data-engine]
created: 2026-04-14
updated: 2026-04-15
sources: []
local_paper: papers/fundamentals/kirillov_2023_sam.pdf
url: https://arxiv.org/abs/2304.02643
status: stable
---

📄 [Full paper](../../papers/fundamentals/kirillov_2023_sam.pdf) · [arXiv](https://arxiv.org/abs/2304.02643) · [Project](https://segment-anything.com)

## TL;DR

Kirillov et al. (Meta AI, ICCV 2023) introduce the **Segment Anything Model (SAM)** — a promptable segmentation foundation model — and the SA-1B dataset (11M images, 1.1B masks) produced by a three-stage data engine. SAM generalizes zero-shot to unseen tasks and domains.

## Problem

Semantic and instance segmentation models had historically been **task-specific** and domain-specific. Vision lacked a prompt-based foundation model analogous to NLP LLMs that could perform new segmentation tasks zero-shot.

## Method

- **Task**: promptable segmentation — given an image and a prompt (point, box, rough mask, free-form text embedding), produce a valid mask. "Valid" under ambiguity = output multiple plausible masks.
- **Architecture**:
  - Heavyweight image encoder (ViT-H) — run once per image, produces an embedding.
  - Lightweight prompt encoder for points/boxes/masks/text.
  - Mask decoder (2-way transformer) predicts ≤3 masks + a confidence score.
- **Data engine** (3 stages): assisted-manual → semi-automatic → fully automatic. Final stage generates masks from a dense grid of point prompts, culls by IoU + stability.
- **SA-1B**: 11M diverse licensed images, 1.1B masks — largest mask dataset released.

## Results

- Zero-shot mask quality: comparable to or better than fully-supervised specialist models (e.g. DeepMask, PolygonRNN++).
- Competitive on 23 segmentation datasets spanning medical, satellite, microscopy — without fine-tuning.
- Inference: <50 ms per prompt after the image encoder pass (which runs in ~0.15 s on GPU).

## Why it matters

SAM is the **default mask source** for 2024–2026 object-centric 3D work: background removal, instance-aware radiance fields, segmented 3DGS editing, and open-vocabulary 3D understanding (often paired with [[clip|CLIP]] or DINO features). Established that high-quality masks are a commoditized upstream primitive.

## Pipeline contribution

- **Promptable segmentation + ambiguity-aware multi-mask head (N1)** — point/box/text prompt → ≤3 valid masks + confidence. candidate thread: [[open-vocab-2d-composition]] Pipeline A · stage: *mask quality* · replaces/augments: *Mask R-CNN / class-specific mask heads* · expected gain: clean boundaries snapped to any spatially-coherent region score — the piece Trident relies on. Also superseded-by [[carion2026_sam-3]] in Pipeline B.
- **Per-image encode-once / per-prompt decode architecture (N2)** — heavyweight ViT-H image encoder run once, lightweight decoder per prompt. candidate thread: [[lifting-foundation-models-to-3d]] · stage: *2D mask generation for 3D lifting* · expected gain: amortized cost across many object-queries per scene — the architecture that makes per-Gaussian identity distillation (Gaussian Grouping, LangSplat) tractable.
- **SA-1B data engine (N3)** — three-stage (assisted→semi→fully automatic) mask generation at scale. Not a pipeline component, but it is the foundation [[carion2026_sam-3]]'s SA-Co engine explicitly builds on. Thread-level evidence that high-quality masks are a commodity input.

## Relation to prior work

- Precursor to video-extended SAM 2 (2024).
- Commonly composed with [[clip|CLIP]] (Grounded-SAM) for text-driven prompting.
- Powers segmentation-aware 3D reconstruction (e.g. Gaussian Grouping, GaussianEditor).

## Open questions / limitations

- No semantic labels — SAM segments "things" but doesn't name them; downstream text alignment required.
- Ambiguity handling via 3 outputs is a heuristic, not a principled probabilistic formulation.
- Fails on very fine structures (hair, thin wires) and some medical imaging modalities without fine-tuning.

## References added to the wiki

- [[sam]] (stub expanded).
