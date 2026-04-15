---
title: "Gaussian Grouping: Segment and Edit Anything in 3D Scenes"
type: paper
tags: [3dgs, segmentation, scene-editing, sam-lifting]
created: 2026-04-15
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/ye_2024_gaussian-grouping.pdf
url: https://arxiv.org/abs/2312.00732
status: stable
---

📄 [Full paper](../../papers/radiance-fields/ye_2024_gaussian-grouping.pdf) · [arXiv](https://arxiv.org/abs/2312.00732) · [GitHub](https://github.com/lkeab/gaussian-grouping)

## TL;DR

Ye, Danelljan, Yu & Ke (ECCV 2024) extend [[3d-gaussian-splatting|3DGS]] with a per-Gaussian **Identity Encoding** supervised by [[sam|SAM]] 2D masks across views + a 3D spatial-consistency regularizer. Result: the splat simultaneously reconstructs appearance and **segments "anything"** at the instance/stuff level in 3D, enabling downstream object removal, inpainting, colorization, style transfer, and scene recomposition.

## Problem

3DGS models appearance and geometry superbly but is **oblivious to object structure** — no per-Gaussian notion of "which instance does this belong to." Prior NeRF-based 3D scene-understanding methods required manually labeled datasets or scanned point clouds, limiting generalization. A scalable recipe for lifting 2D instance masks into 3D was missing.

## Method

- **Identity Encoding**: augments each Gaussian with a compact learnable vector (low-dim feature) indicating its group membership.
- **Supervision**: render the Identity field like color; compare to 2D SAM masks across views via cross-entropy per tracked mask ID.
- **Cross-view mask association**: SAM masks are associated across frames via IoU + feature similarity so a single "track" supervises a consistent 3D identity.
- **3D spatial-consistency regularizer**: Gaussians in local neighborhoods should share identity unless depth/normal changes indicate a boundary — suppresses floaty, inconsistent identities.
- **Local Gaussian Editing**: because identities are stored per-Gaussian, editing operations (remove, recolor, restyle, move) can be targeted at specific groups without retraining.

## Results

- Cleanly segments open-world 3D scenes without any 3D labels.
- Downstream edits (object removal + inpainting, colorization, style transfer, object relocation/recomposition) work with minimal artifacts.
- Faster, sharper, and more edit-friendly than NeRF-based 3D-segmentation predecessors.

## Why it matters

Canonical demonstration of the **"lift 2D foundation-model output to per-Gaussian attribute"** pattern. Opens the door to:
- Per-object 3D editing pipelines without retraining.
- Downstream open-vocabulary concept grounding when paired with [[clip|CLIP]] features ([[qin2024_langsplat|LangSplat]], [[jiao2025_clip-gs|CLIP-GS]]).
- In-the-wild extensions with robust per-Gaussian features ([[bao2025_seg-wild|Seg-Wild]]).

## Relation to prior work

- Extends [[3d-gaussian-splatting]] with an identity channel.
- 3D counterpart to [[kirillov2023_sam|SAM]] — uses SAM as the supervisor, not as an inference-time module.
- Contrasts with NeRF-based 3D segmentation (NeRF-SOS, Panoptic Lifting): explicit Gaussian representation makes identities addressable for editing.

## Open questions / limitations

- Mask-association across views is heuristic; long-horizon or occlusion-heavy scenes still produce identity fragmentation.
- Identity dimensionality is a free hyperparameter; no principled choice for scene complexity.
- No inherent language alignment — needs a downstream CLIP/vision-language step (supplied by LangSplat, CLIP-GS).

## References added to the wiki

- [[3d-gaussian-splatting]] (cross-link), [[sam]] (lineage).
