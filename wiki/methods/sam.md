---
title: Segment Anything (SAM)
type: method
tags: [segmentation, foundation-model, promptable]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/kirillov2023_sam.md, wiki/papers/carion2026_sam-3.md, wiki/papers/chen2025_sam-3d.md, wiki/papers/ye2024_gaussian-grouping.md, wiki/papers/qin2024_langsplat.md, wiki/papers/bao2025_seg-wild.md]
status: draft
---

## What it is

SAM (Kirillov et al. 2023, Meta AI) is a **promptable segmentation foundation model**: given an image and a prompt (point, box, mask, or text via CLIP), it emits high-quality segmentation masks without per-dataset training.

Part of the [[foundation-model]] family. Defines the **promptable** row of the [[segmentation]] task taxonomy; composes with [[clip|CLIP]] + [[dinov2|DINO]] in [[open-vocab-2d-composition]].

## Architecture
- Heavyweight image encoder (ViT) computed once per image.
- Lightweight prompt encoder + mask decoder producing multiple mask hypotheses per prompt.
- Trained on the SA-1B dataset (1B masks, 11M images) via a data engine.

## Why it matters in photogrammetry
- Pre-processing for object-centric 3D reconstruction (masking backgrounds, isolating instances).
- Supervision signal for open-vocabulary and instance-aware radiance fields / 3DGS.
- Backbone for the "segmented Gaussian Splatting" line of work — see [[lifting-foundation-models-to-3d]]:
  - [[ye2024_gaussian-grouping|Gaussian Grouping]] (per-Gaussian SAM-supervised identities)
  - [[qin2024_langsplat|LangSplat]] (SAM-hierarchy supervision of CLIP language fields)
  - [[bao2025_seg-wild|Seg-Wild]] (SAM-driven refinement for in-the-wild 3DGS segmentation)

## Variants & lineage
- SAM (2023) → SAM 2 (video, 2024) → [SAM 3](../papers/carion2026_sam-3.md) (2026) — Promptable Concept Segmentation with text + image exemplars.
- Sibling: [SAM 3D](../papers/chen2025_sam-3d.md) (2025) — single-image 3D reconstruction under the Segment Anything umbrella.
- Also: Grounded-SAM (language-driven prompting, CLIP + SAM composition).

## Key references
- [Kirillov et al. 2023](../papers/kirillov2023_sam.md) · [pdf](../../papers/fundamentals/kirillov_2023_sam.pdf) — canonical SAM paper, ICCV 2023.
