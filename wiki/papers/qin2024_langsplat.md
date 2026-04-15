---
title: "LangSplat: 3D Language Gaussian Splatting"
type: paper
tags: [3dgs, open-vocabulary, clip, language-field, sam]
created: 2026-04-15
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/qin_2024_langsplat.pdf
url: https://arxiv.org/abs/2312.16084
status: stable
---

📄 [Full paper](../../papers/radiance-fields/qin_2024_langsplat.pdf) · [arXiv](https://arxiv.org/abs/2312.16084) · [Project](https://langsplat.github.io)

## TL;DR

Qin, Li, Zhou, Wang & Pfister (CVPR 2024) construct a **3D language field on [[3d-gaussian-splatting|3DGS]]** by distilling [[clip|CLIP]] features into per-Gaussian language tokens — replacing the NeRF-based LERF. Uses [[sam|SAM]] to learn **hierarchical semantics** (subpart / part / whole), eliminating LERF's multi-scale query costs. **~199× faster** than LERF at 1440×1080 with substantially sharper boundaries.

## Problem

LERF (Kerr et al. 2023) grounded CLIP features in NeRF to enable open-ended language queries in 3D, but suffered two failures: (1) **blurry boundaries** from NeRF's volumetric averaging + CLIP's image-level supervision, and (2) **expensive multi-scale queries** — CLIP operates at whole-image granularity, so LERF had to render at many scales and regularize with DINO. Query time was prohibitive.

## Method

- **Replace NeRF with 3DGS**: tile-based splatting is orders of magnitude faster at rendering language features than volumetric sampling.
- **Per-scene language autoencoder**: CLIP embeddings are 512-D; per-Gaussian storage is expensive. LangSplat first trains a compact scene-specific autoencoder, then stores the low-D latent per Gaussian and decodes at query time.
- **SAM-hierarchical supervision**: [[sam|SAM]] produces mask hierarchies (whole / part / subpart). Extract CLIP embeddings at each hierarchy level and distill into the 3DGS language field at corresponding scales — removes the need for image-scale sweeps at query time.
- **Dropped DINO regularization**: hierarchy supervision from SAM + explicit 3DGS geometry provide the spatial coherence DINO was adding.

## Results

- Outperforms LERF by a large margin on 3D-OVS and LERF benchmarks.
- **199× speedup** at 1440×1080 rendering.
- Sharp object-level boundaries on open-vocabulary queries ("the leather jacket", "the red mug").

## Why it matters

LangSplat **is the canonical CLIP-lifted-to-3DGS method** and the immediate baseline for any 2D→3D language grounding work. Establishes the three-ingredient recipe reused across the field:
1. Replace NeRF with 3DGS for speed.
2. Compress language embeddings via a latent autoencoder.
3. Use SAM's mask hierarchy as a free structural prior.

## Pipeline contribution

- **CLIP distillation into per-Gaussian latents via scene-specific autoencoder (N1)** — 512-D CLIP → low-D latent per Gaussian; decoded at query time. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline I · stage: *per-primitive feature storage* · replaces/augments: *raw 512-D per Gaussian (memory explosion) / LERF's multi-scale MLP* · expected gain: 199× speedup at 1440×1080 + sharper boundaries.
- **SAM-mask-hierarchy supervision (N2)** — whole/part/subpart CLIP embeddings distilled at corresponding scales; removes LERF's multi-scale query sweeps. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline I · stage: *multi-scale semantic supervision* · replaces/augments: *image-scale sweeps at query time* · expected gain: hierarchy information baked in at training; free at query time.
- **Dropped DINO regularization (N3)** — 3DGS geometry + SAM hierarchy together supply the spatial coherence DINO provided in LERF. candidate thread: [[lifting-foundation-models-to-3d]] · stage: *regularization* · reverse-engineering note: the thread's synthesis bet *"re-add DINOv3 regularization to LangSplat"* tests whether this drop was warranted at DINOv2 or only at DINOv3.
- **Role**: LangSplat is the Pipeline-I SOTA recipe and the immediate baseline for every subsequent CLIP-lifting paper.

## Relation to prior work

- NeRF predecessor: LERF (2023).
- Contemporaneous 3DGS segmentation sibling: [[ye2024_gaussian-grouping|Gaussian Grouping]] (identity-only, no language).
- Successor direction: [[jiao2025_clip-gs|CLIP-GS]] (contrastive unification instead of distillation), [[bao2025_seg-wild|Seg-Wild]] (extends to in-the-wild collections).

## Open questions / limitations

- Per-scene autoencoder means no cross-scene transfer of the latent space.
- CLIP's still-coarse semantics limit fine-grained open-vocab queries on visually similar classes.
- SAM-hierarchy supervision inherits SAM's failure modes (thin structures, transparent objects).

## References added to the wiki

- [[3d-gaussian-splatting]], [[clip]], [[sam]] — cross-links + lineage.
