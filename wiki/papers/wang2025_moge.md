---
title: "MoGe: Unlocking Accurate Monocular Geometry Estimation for Open-Domain Images with Optimal Training Supervision"
type: paper
tags: [monocular-depth, mde, affine-invariant, foundation-model]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/mvs-depth/wang_2025_moge.pdf
url: https://arxiv.org/abs/2410.19115
code: https://github.com/microsoft/MoGe
license_paper: arxiv-nonexclusive
license_code: MIT
status: stub
---

📄 [arXiv](https://arxiv.org/abs/2410.19115) · [code](https://github.com/microsoft/MoGe)

_Paper license: `arxiv-nonexclusive` · Code license: `MIT`_

## TL;DR
Monocular geometry foundation model: predicts affine-invariant point maps from single open-domain images. Training supervision is designed to isolate scale + shift ambiguity so downstream consumers can fit `α, β` cleanly. CVPR 2025. The preferred MDE model in [[yu2025_madpose]]'s best calibrated-pose results.

## Why it's in the wiki
- Load-bearing MDE prior for [[yu2025_madpose]]: Ours-calib + MoGe + SP+LG gives the best calibrated-pose AUC@10° on ScanNet-1500 (42.18 vs 39.11 for PoseLib-5pt baseline).
- Alternative to Depth-Anything v2 / Marigold / Omnidata in the [[mono-depth-estimation]] thread. Specifically tuned for affine-invariant geometry (not metric), making it a natural pair for affine-correcting solvers like [[affine-corrected-minimal-relative-pose-solvers_yu2025]].

## Status
Stub — cited as MDE prior in one paper page. Expand if a future ingest studies its mechanism (optimal training supervision for affine-invariant geometry) in depth.
