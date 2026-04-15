---
title: "ConceptFusion: Open-set Multimodal 3D Mapping"
type: paper
tags: [voxel, tsdf, clip, dino, foundation-model, open-set, robotics, scene-understanding]
created: 2026-04-15
updated: 2026-04-15
sources: []
local_paper: papers/radiance-fields/jatavallabhula_2023_conceptfusion.pdf
url: https://arxiv.org/abs/2302.07241
code: https://github.com/concept-fusion/concept-fusion
license_code: MIT
license_paper: CC-BY-SA-4.0
status: stable
---

📄 [Full paper](../../papers/radiance-fields/jatavallabhula_2023_conceptfusion.pdf) · [arXiv](https://arxiv.org/abs/2302.07241) · [Project](https://concept-fusion.github.io) · [code](https://github.com/concept-fusion/concept-fusion)

_Code license: `MIT`_

## TL;DR

Jatavallabhula et al. (MIT + collaborators, RSS 2023) build **open-set multimodal 3D maps** by fusing pixel-aligned [[foundation-model|foundation-model]] features ([[clip|CLIP]], [[dinov2|DINO]], AudioCLIP) into a **voxelized 3D representation** via classical [[tsdf|TSDF]]-style integration over RGB-D streams. The resulting maps support zero-shot queries by **text, image, audio, or click** — all concepts share the same embedding space. The canonical voxel-based counterpart to the 3DGS-lifting papers.

## Problem

2023-era semantic 3D mapping was confined to **closed-set** categories and text-only queries. Methods like LERF distilled CLIP into NeRFs but needed per-scene training and offered only language queries. Robots operating in novel environments needed **open-set, multimodal, online** mapping — build the map as you go, query it by any modality later.

## Method

- **Pixel-aligned open-set features**: for each input RGB frame, use [[sam|SAM]] (or a class-agnostic mask generator) to extract per-object masks; run CLIP/DINO/AudioCLIP on crops; project per-pixel back to the image — yielding a dense feature map where each pixel carries its object's foundation-model embedding.
- **Voxel-hashed TSDF fusion**: standard voxel-hashing 3D mapping (à la KinectFusion / Voxblox) — but each voxel carries not just SDF + weight but also a **running weighted average of foundation-model features**.
- **Multimodal alignment**: because CLIP aligns text+image+audio in one embedding space (AudioCLIP extending it), a query in any modality → its CLIP/AudioCLIP embedding → cosine similarity to per-voxel features → relevant 3D regions.
- **3D spatial reasoning**: modules on top of the voxel map compute distances, between-objects relationships, and aggregate for higher-level queries ("how far is the TV from the fridge?").
- **Online**: integration happens per-frame; maps are queryable immediately without offline training.

## Results

- Strong retention of **fine-grained concepts** (Disney characters, specific objects) that closed-set baselines miss entirely.
- Robotic manipulation: zero-shot tabletop rearrangement driven by language queries.
- Autonomous driving demo: "drive me to a roundabout" → map localization and navigation.
- Robust across SceneNN, Replica, and real RGB-D captures.

## Why it matters

**The voxel lane of [[lifting-foundation-models-to-3d]]**. Where [[qin2024_langsplat|LangSplat]] and [[jiao2025_clip-gs|CLIP-GS]] lift CLIP onto 3DGS primitives per-scene, ConceptFusion fuses features into a **classical voxel-TSDF map** online — trading neural rendering for robotics-grade speed and multimodality. Establishes that the lifting pattern is representation-agnostic: voxel, Gaussian, or implicit all admit the same treatment.

Key differentiators vs. the 3DGS lane:
- **Online / no per-scene training** — features integrate frame-by-frame like classical SLAM.
- **Multimodal** (image + text + audio + click) — most 3DGS methods stop at text.
- **Robotics-ready** — voxel grids interface naturally with planners, occupancy checks, and ray-casting.
- **Loses photorealistic rendering** — no novel-view synthesis, just semantic mapping.

## Pipeline contribution

- **Per-pixel open-set feature generation via SAM-mask crops + CLIP/DINO/AudioCLIP (N1)** — mask region → crop → FM encode → project per-pixel. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline IV · stage: *2D feature extraction for online fusion* · replaces/augments: *closed-set label maps* · expected gain: open-set + multimodal queries from a single fusion pipeline.
- **Voxel-hashed TSDF fusion extended with per-voxel running feature average (N2)** — classical KinectFusion weighting applied to FM features. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline IV · stage: *online geometry + feature fusion* · replaces/augments: *two-pass (geometry, then features)* · expected gain: one-pass online mapping; robotics-grade latency.
- **Modality-unified querying (text/image/audio/click) (N3)** — CLIP/AudioCLIP shared embedding space → single cosine query routes all modalities. candidate thread: [[lifting-foundation-models-to-3d]] · stage: *query-time multimodal retrieval* · expected gain: the only lifting pipeline that handles audio queries natively.
- **Synthesis-bet contribution**: ConceptFusion's voxel-fusion recipe + [[heinrich2025_radiov25]] RADIOv2.5 as the unified teacher + [[deng2026_vpgs-slam]]'s progressive submaps yields an online city-scale multimodal lifted 3D — the "Pipeline IV++" bet flagged in the thread.

## Relation to prior work

- Inherits the voxel-TSDF pipeline from [[curless1996_tsdf|Curless & Levoy 1996]] and KinectFusion.
- Contrast with LERF (neural radiance field + CLIP) — ConceptFusion is explicit voxels + CLIP + multimodality.
- 3DGS counterparts (same era +1): [[ye2024_gaussian-grouping|Gaussian Grouping]], [[qin2024_langsplat|LangSplat]], [[jiao2025_clip-gs|CLIP-GS]], [[bao2025_seg-wild|Seg-Wild]].
- Sparse-voxel successor: [[wu2026_langsvr|LangSVR (Wu 2026)]] adds feature field distillation + geometric priors to sparse voxels.

## Open questions / limitations

- Voxel resolution trades memory vs. detail; scene-scale captures require voxel-hashing or octrees to stay tractable.
- No novel-view synthesis — purely a semantic map, not a rendering representation.
- Feature fusion is a per-voxel running average; no disentanglement of view-dependent vs. view-invariant semantics.
- Mask generator quality upstream bounds final feature quality.

## In the wiki
- Central evidence for the voxel-representation branch of [[lifting-foundation-models-to-3d]].
- Connects [[tsdf]] (classical fusion) with [[foundation-model]] (modern semantics).
