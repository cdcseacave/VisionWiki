---
title: "LoFTR: Detector-Free Local Feature Matching with Transformers"
type: paper
tags: [feature-matching, detector-free, transformer, semi-dense, coarse-to-fine]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/feature-matching/sun_2021_loftr.pdf
url: https://arxiv.org/abs/2104.00680
code: https://github.com/zju3dv/LoFTR
license_paper: arxiv-nonexclusive
license_code: Apache-2.0
status: stub
---

📄 [arXiv](https://arxiv.org/abs/2104.00680) · [code](https://github.com/zju3dv/LoFTR)

_Paper license: `arxiv-nonexclusive` · Code license: `Apache-2.0`_

## TL;DR
Semi-dense detector-free image matcher: skip keypoint detection, produce dense correspondences directly by running self- and cross-attention over coarse (1/8 stride) feature maps, then refine matches at subpixel resolution via a fine-level transformer. Standard matcher reference for the detector-free family (LoFTR, MatchFormer, AspanTransformer).

## Why it's in the wiki
- The feature-matching frontend in [[coarse-to-fine-detector-free-sfm-bridge_he2023]]: DetectorFreeSfM feeds LoFTR's coarse-stride matches (at `r=8`) directly into COLMAP-style incremental mapping, bypassing detector repeatability failures.
- One of the matchers supported by the MADPose hybrid estimator pipeline ([[yu2025_madpose]]).
- Historical: LoFTR established that attention-based dense matching outperforms detect-then-match on texture-poor scenes, seeding the detector-free family.

## Status
Stub — cited by two paper pages. Expand to a full page if a future ingest relies on LoFTR's mechanism details.
