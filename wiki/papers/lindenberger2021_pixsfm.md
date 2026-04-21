---
title: "Pixel-Perfect Structure-from-Motion with Featuremetric Refinement"
type: paper
tags: [sfm, feature-metric-ba, track-refinement, pixsfm]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/sfm-slam/lindenberger_2021_pixsfm.pdf
url: https://arxiv.org/abs/2108.08291
code: https://github.com/cvg/pixel-perfect-sfm
license_paper: arxiv-nonexclusive
license_code: Apache-2.0
status: stub
---

📄 [arXiv](https://arxiv.org/abs/2108.08291) · [code](https://github.com/cvg/pixel-perfect-sfm)

_Paper license: `arxiv-nonexclusive` · Code license: `Apache-2.0`_

## TL;DR
Refines 2D keypoint locations and poses inside COLMAP SfM by minimizing a **feature-metric** loss (distance in a learned CNN feature space) rather than a pure geometric reprojection loss. Two phases: pre-BA keypoint-adjustment over feature gradients, post-BA feature-metric BA over the full scene. Reference filler for [[sfm.feature-track-refinement]].

## Why it's in the wiki
- The direct baseline that [[multi-view-transformer-track-refinement_he2023]] and [[iterative-ba-plus-track-topology-adjustment_he2023]] beat: DetectorFreeSfM matches PixSfM accuracy on detector-based inputs and wins by ~3 orders of magnitude less memory on large-scale scenes (Table 4 in [[he2023_detector-free-sfm]]: 0.37 GB vs 904.5 GB on 2000 images) because its geometric BA operates only on refined 2D locations, not feature patches.
- Historical SOTA filler for feature-metric SfM refinement; any thread adopting `sfm.feature-track-refinement` with detector-based inputs defaults to PixSfM.

## Status
Stub — cited as baseline in one paper page. Expand when a bet or design specifically reuses PixSfM's feature-metric BA formulation.
