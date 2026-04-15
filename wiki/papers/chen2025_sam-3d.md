---
title: "SAM 3D: 3Dfy Anything in Images"
type: paper
tags: [single-image-3d, generative-3d, flow-matching, scene-reconstruction]
created: 2026-04-14
updated: 2026-04-14
sources: []
local_paper: papers/mesh-reconstruction/chen_2025_sam-3d.pdf
url: https://ai.meta.com/sam3d
status: stable
---

📄 [Full paper](../../papers/mesh-reconstruction/chen_2025_sam-3d.pdf) · [Project](https://ai.meta.com/sam3d) · [GitHub](https://github.com/facebookresearch/sam-3d-objects) · [Demo](https://www.aidemos.meta.com/segment-anything/editor/convert-image-to-3d)

## TL;DR

SAM 3D Team (Chen, Chu, Gleize, Liang, Sax, Tang, Wang et al.; Meta, 2025) present a **generative single-image 3D reconstruction** system that predicts per-object **geometry, texture, and scene layout** from one natural image. Combines a human/model-in-the-loop annotation pipeline with a two-stage latent flow-matching architecture. Claims ≥5:1 win rate vs. recent work in human preference tests.

## Problem

Single-image 3D reconstruction methods had historically struggled on natural images: heavy occlusion, cluttered scenes, and context-dependent recognition broke methods trained mostly on isolated synthetic objects. Ground-truth 3D is expensive to obtain at scale, creating a "3D data barrier" that limited learned methods.

## Method

- **Task**: single RGB image → composable 3D scene (per-object geometry + texture + layout). Goes beyond isolated-object reconstruction to full scene composition.
- **Data engine**: human-and-model-in-the-loop pipeline for annotating shape, texture, and pose on real-world images at "unprecedented scale" — the paper's core contribution.
- **Architecture**: two-stage latent flow-matching (building on Xiang et al. 2025):
  - Shape generator produces a latent, decoded to geometry + texture.
  - Layout module predicts object placement in the scene.
- **Training**: multi-stage — synthetic pretraining followed by real-world alignment via the annotated data. Breaks the data barrier by combining synthetic scale with real-image distribution.

## Results

- ≥5:1 human preference win rate vs. recent SOTA single-image 3D methods on real-world objects and scenes.
- Promises open release: code, weights, online demo, and a new in-the-wild 3D benchmark.

## Why it matters

Pushes single-image 3D from "isolated synthetic object" territory into **natural-image scene reconstruction**. Relevant for:
- 3D asset creation pipelines (consumer apps, AR/VR).
- Priors for sparse-view or uncalibrated reconstruction.
- Complementing SfM/MVS where coverage is insufficient (single-image gaps can be hallucinated in).

## Pipeline contribution

- **Two-stage latent flow-matching for single-image 3D scene (N1)** — shape generator produces latent → geometry + texture; layout module predicts placement. candidate thread: [[lifting-foundation-models-to-3d]] Pipeline VII · stage: *single-image generative 3D* · replaces/augments: *isolated-object 3D generators* · expected gain: ≥5:1 human preference win rate vs prior SOTA on real-world objects + scenes.
- **Human/model-in-the-loop data engine for real-image 3D annotation (N2)** — breaks the "3D data barrier" by combining synthetic pretraining with real-image alignment. candidate thread: *3D generative foundations* · stage: *training data pipeline* · expected gain: generalizes from isolated objects to cluttered natural images where prior methods fail.
- **Per-object geometry + texture + scene layout as unified output (N3)** — composable 3D scene, not just isolated object. candidate thread: [[lifting-foundation-models-to-3d]] · stage: *scene composition* · expected gain: downstream use in asset creation + sparse-view-reconstruction priors.
- **Role**: SAM 3D is the **single-image generative lane** of the thread — Pipeline VII; orthogonal to every per-scene-distillation method (I–V). Naming suggests future integration with SAM 3 ([carion2026_sam-3]) masks as input conditioning — untried synthesis bet.

## Relation to prior work

- Builds on two-stage latent flow-matching (Xiang et al. 2025) — extends from isolated objects to scene-level layout.
- Orthogonal to [[dust3r|DUSt3R]]/[[mast3r|MASt3R]]/[[vggt|VGGT]] which need ≥2 views; SAM 3D operates from a single image.
- Part of Meta's Segment Anything "3D" branch — naming suggests future integration with [[carion2026_sam-3|SAM 3]] masks as input.

## Open questions / limitations

- No arXiv preprint at time of ingest — reference paper distributed via Meta AI blog/project site.
- Generative hallucination vs. fidelity: single-image 3D is fundamentally underconstrained; paper does not quantify the reconstruction-vs-generation trade.
- No discussion of compositing reconstructed objects into existing scenes (texture/lighting consistency).

## References added to the wiki

- [[gaussian-to-mesh-pipelines]] thread — orthogonal single-image paradigm to note.


## In the wiki
- Central evidence in [[lifting-foundation-models-to-3d]] — single-image generative 3D under the SAM umbrella.
