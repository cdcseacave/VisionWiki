---
title: Segmentation (task taxonomy)
type: concept
tags: [segmentation, semantic, instance, panoptic, promptable, open-vocabulary]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/kirillov2023_sam.md, wiki/papers/carion2026_sam-3.md, wiki/papers/shi2024_open-vocab-segmentation.md]
status: draft
---

## What it is

Segmentation is the assignment of a label (class, instance, or group) to each pixel (2D) or primitive (3D Gaussian, voxel, point). The field has splintered into several distinct *tasks* that are often conflated; this page disambiguates them.

## Task taxonomy

| Task | Input | Output | Canonical method |
|---|---|---|---|
| **Semantic** | Image | Per-pixel class label from a fixed vocabulary | FCN, DeepLab, SegFormer |
| **Instance** | Image | Per-pixel class + instance ID (no background classification) | Mask R-CNN, Mask2Former |
| **Panoptic** | Image | Per-pixel class + instance ID, covering *every* pixel (things + stuff) | Panoptic-DeepLab, Mask2Former |
| **Promptable** | Image + prompt (point/box/mask/text) | Mask(s) for the specified region/concept | [[sam|SAM]] · [SAM 2](../papers/kirillov2023_sam.md) |
| **[[open-vocabulary-segmentation\|Open-vocabulary]]** | Image + free-form text | Mask(s) for any concept specified at inference | [Trident](../papers/shi2024_open-vocab-segmentation.md) · [SAM 3](../papers/carion2026_sam-3.md) |

**Key axes**:
- **Class vocabulary**: closed (fixed set) vs. open (text-specified at inference).
- **Identity**: class-only (semantic) vs. class + instance (instance / panoptic).
- **Interactivity**: automatic vs. prompt-driven (promptable).
- **Training supervision**: labeled masks per class vs. masks-only (SAM) vs. image-text pairs + masks (SAM 3, Trident).

## 3D segmentation

Lifting 2D segmentation into 3D uses two main strategies:
- **Per-primitive attribute**: augment each Gaussian / voxel with an identity or feature channel, supervise via rendered 2D masks across views. See [[ye2024_gaussian-grouping|Gaussian Grouping]] (identity) and [[qin2024_langsplat|LangSplat]] (CLIP features).
- **Mask voting / consensus**: run 2D segmentation per view, aggregate across views via 3D consistency (KNN/graph smoothing).

See [[lifting-foundation-models-to-3d]] for the synthesis.

## Why the taxonomy matters

Papers casually say "segmentation" and mean different things. Open-vocab papers often compare to promptable baselines (and vice versa), muddying benchmarks. When reading a new segmentation paper, first pin down which row of the table it lives in.

## Key references
- [Kirillov et al. 2023 (SAM)](../papers/kirillov2023_sam.md) — defines promptable segmentation.
- [Carion et al. 2026 (SAM 3)](../papers/carion2026_sam-3.md) — unifies promptable + open-vocabulary via concept prompts.
- [Shi et al. 2024 (Trident)](../papers/shi2024_open-vocab-segmentation.md) — training-free open-vocab via foundation-model composition.
