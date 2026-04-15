---
title: "SAM 3: Segment Anything with Concepts"
type: paper
tags: [segmentation, foundation-model, open-vocabulary, video-tracking]
created: 2026-04-14
updated: 2026-04-15
sources: []
local_paper: papers/fundamentals/carion_2026_sam-3.pdf
url: https://arxiv.org/abs/2511.16719
code: https://github.com/facebookresearch/sam3
license_code: SAM License
license_paper: arxiv-nonexclusive
status: stable
---

📄 [Full paper](../../papers/fundamentals/carion_2026_sam-3.pdf) · [arXiv](https://arxiv.org/abs/2511.16719) · [Project](https://ai.meta.com/sam3) · [code](https://github.com/facebookresearch/sam3)

_Code license: `SAM License`_

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

## Pipeline contribution

- **Promptable Concept Segmentation (PCS) — unified text/exemplar-prompted masks + instance IDs + tracking (N1)** — collapses Pipeline A's separate CLIP + SAM stages into one model. candidate thread: [[open-vocab-2d-composition]] Pipeline B · stage: *full pipeline in one forward pass* · replaces/augments: *Trident's three-backbone composition* · expected gain: 2× accuracy over prior open-vocab segmenters on image/video PCS, plus video-native tracking.
- **Presence head (N2)** — decouples "concept present at all" from per-instance localization. candidate thread: [[open-vocab-2d-composition]] · stage: *false-positive suppression* · replaces/augments: *per-detection thresholding* · expected gain: removes hallucinated detections on absent concepts — a consistent failure mode in the Trident and GroundingDINO lineage.
- **SA-Co data engine (N3)** — 4M concept labels with hard-negative mining. Thread-level evidence; reinforces [[kirillov2023_sam]]'s claim that data-engine curation is the scaling axis for segmentation foundation models.
- **Lift to 3D is not addressed in the paper** — candidate thread: [[lifting-foundation-models-to-3d]] · stage: *2D mask source with instance IDs* · candidate: *SAM 3 masks → per-Gaussian identity supervision in the Gaussian Grouping recipe; the instance IDs make cross-view association free.* No paper does this yet — this is the canonical downstream synthesis bet. Expected gain: kills Gaussian Grouping's "cross-view association" module (its most fragile stage) by having IDs already consistent out of the box.

## Relation to prior work

- Direct successor to [[sam|SAM]] (2023) and SAM 2 (video, 2024).
- Unifies concept detection (GroundingDINO lineage) with mask quality (SAM).
- Pairs naturally with [[clip|CLIP]]/SigLIP for text-side prompting.

## Open questions / limitations

- Concept definition is still fuzzy at the edges — compositional prompts ("red chair *without* armrests") untested.
- Video memory is bounded; very long-horizon identity preservation across occlusions unclear.
- Paper does not explicitly address 3D; lifting-to-3D is left for downstream work.

## Code & license

Repo ships under Meta's SAM License (custom, NOASSERTION on GitHub) — non-commercial research use only; blocks redistribution in commercial pipelines without separate licensing.

## References added to the wiki

- [[sam]] method page — lineage updated.


## In the wiki
- Unifies [[open-vocabulary-segmentation|OVS]] and promptable segmentation via Promptable Concept Segmentation.
- See [[open-vocab-2d-composition]] for the pattern it subsumes.
