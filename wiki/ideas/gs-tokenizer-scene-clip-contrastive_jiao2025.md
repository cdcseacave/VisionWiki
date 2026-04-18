---
title: GS Tokenizer + scene-level CLIP contrastive alignment (CLIP-GS)
type: idea
source_paper: wiki/papers/jiao2025_clip-gs.md
also_in: []

scope: new-paradigm
stages: [lifting-foundation-models.3dgs-tokenization, lifting-foundation-models.scene-level-alignment]
collapses: []
splits_into: []
rewrites: {}

inputs: [pre-computed-3dgs-scene, rendered-images, text-labels]
outputs: [scene-level-embedding-aligned-with-clip]
assumptions: [3dgs-already-computed, scene-level-triplets-available, object-to-scene-scale-works]
requires_upstream_property: [3dgs-pretrained]
requires_downstream_property: [consumer-uses-scene-level-retrieval-or-classification]
learned_params: [gs-tokenizer-weights, transformer-encoder-weights]
failure_modes: [scene-scale-tokenization-strained, serialization-order-heuristic, 3dgs-must-be-precomputed]

requires: []
unlocks: []
co_requires: [pointcloud-fm-warm-start_jiao2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, clip, contrastive-alignment, zero-shot-transfer]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**GS Tokenizer**: serialize 3DGS into a token stream via farthest-point-sampling centers + k-NN patches that capture position, color, scale, rotation, opacity. **Transformer encoder** (warm-started from point-cloud FM — Uni3D/OpenShape) produces a scene-level embedding. **Symmetric InfoNCE** contrastive loss on (3DGS, rendered images, text) triplets. **Image-voting loss**: multi-view rendered images vote on gradient direction → rotation/pose-invariant embeddings.

## Why it wins

Opposite axis-2 choice from LangSplat's per-scene distillation: generalizable across scenes zero-shot. SOTA retrieval + zero-shot + few-shot over ULIP/Uni3D/OpenShape (point-cloud-based predecessors). Establishes 3DGS as a first-class multimodal 3D input — not just a rendering substrate.

## Pipeline-shape implications

New-paradigm: the alignment is **scene-level**, not per-primitive. Threads using this replace per-Gaussian feature distillation with a single encoder producing one embedding per scene. Can't mix per-scene-distillation and scene-level alignment on the same scene without architecture changes.

## Open questions

- Scene-scale tokenization (thousands of Gaussians) strained; object-scale works best.
- Serialization-order sensitivity not characterized.
