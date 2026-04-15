---
title: "SAM 3: Segment Anything with Concepts"
type: paper
tags: [segmentation, foundation-model, open-vocabulary, video-tracking]
created: 2026-04-14
updated: 2026-04-14
sources: []
local_paper: papers/fundamentals/carion_2026_sam-3.pdf
url: https://arxiv.org/abs/2511.16719
status: stable
---

📄 [Full paper](../../papers/fundamentals/carion_2026_sam-3.pdf) · [arXiv](https://arxiv.org/abs/2511.16719) · [Project](https://ai.meta.com/sam3)

## TL;DR

Carion et al. (Meta Superintelligence Labs, 2026) introduce **SAM 3**, a unified model for **Promptable Concept Segmentation (PCS)**: given either a short noun phrase ("yellow school bus"), image exemplars, or both, return segmentation masks *and unique instance identities* for all matching objects across images and videos. Doubles accuracy of existing open-vocabulary / concept-driven segmenters.

## Problem

Prior [[sam|SAM]] was promptable but **identity-blind**: a point prompt returned one mask, not "every instance of this concept." Open-vocabulary detectors (GroundingDINO, OWL-ViT) produced boxes and class scores but poor masks and no tracking. No single system reliably did: detect, segment, and track arbitrary user-defined concepts in video.

## Method

- **Task — Promptable Concept Segmentation**: input = image/video + concept prompt (text, image exemplars, or both). Output = masks + unique IDs for all matching instances.
- **Architecture**:
  - Shared backbone (vision encoder).
  - **Image-level detector** branch for per-frame PCS.
  - **Memory-based video tracker** branch for identity propagation.
  - **Presence head**: decouples "is the concept in the image at all" from per-instance localization — a key accuracy driver that addresses the "hallucinated detections on absent concepts" failure mode.
- **Data engine** producing **SA-Co** (Segment Anything with Concepts): 4M unique concept labels across images and videos, with hard-negative mining.

## Results

- **2× accuracy** of prior SOTA on image and video PCS benchmarks.
- Improves on prior SAM capabilities for point/box/mask-prompted segmentation.
- Released with open-source code, weights, and the SA-Co benchmark.

## Why it matters

Consolidates three previously-separate capabilities (open-vocab detection, instance segmentation, video tracking) into one prompt-driven model. For 3D pipelines: enables **concept-conditioned 3D segmentation** (e.g. "segment all chairs in this reconstruction") by lifting SAM 3 masks to 3DGS or NeRF assets.

## Relation to prior work

- Direct successor to [[sam|SAM]] (2023) and SAM 2 (video, 2024).
- Unifies concept detection (GroundingDINO lineage) with mask quality (SAM).
- Pairs naturally with [[clip|CLIP]]/SigLIP for text-side prompting.

## Open questions / limitations

- Concept definition is still fuzzy at the edges — compositional prompts ("red chair *without* armrests") untested.
- Video memory is bounded; very long-horizon identity preservation across occlusions unclear.
- Paper does not explicitly address 3D; lifting-to-3D is left for downstream work.

## References added to the wiki

- [[sam]] method page — lineage updated.


## In the wiki
- Unifies [[open-vocabulary-segmentation|OVS]] and promptable segmentation via Promptable Concept Segmentation.
- See [[open-vocab-2d-composition]] for the pattern it subsumes.
