---
title: Segment Anything (SAM)
type: method
tags: [segmentation, foundation-model, promptable]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/kirillov2023_sam.md]
status: draft
---

## What it is

SAM (Kirillov et al. 2023, Meta AI) is a **promptable segmentation foundation model**: given an image and a prompt (point, box, mask, or text via CLIP), it emits high-quality segmentation masks without per-dataset training.

## Architecture
- Heavyweight image encoder (ViT) computed once per image.
- Lightweight prompt encoder + mask decoder producing multiple mask hypotheses per prompt.
- Trained on the SA-1B dataset (1B masks, 11M images) via a data engine.

## Why it matters in photogrammetry
- Pre-processing for object-centric 3D reconstruction (masking backgrounds, isolating instances).
- Supervision signal for open-vocabulary and instance-aware radiance fields / 3DGS.
- Backbone for the "segmented Gaussian Splatting" line of work.

## Variants & lineage
- SAM → SAM 2 (video, 2024) → Grounded-SAM (language-driven prompting).

## Key references
- [Kirillov et al. 2023](../papers/kirillov2023_sam.md) · [pdf](../../papers/fundamentals/kirillov_2023_sam.pdf) — canonical SAM paper, ICCV 2023.
