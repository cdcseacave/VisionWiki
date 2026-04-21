---
title: "Detector-Free Structure from Motion"
type: paper
tags: [sfm, detector-free, loftr, coarse-to-fine, texture-poor, multi-view-transformer, track-topology]
created: 2026-04-21
updated: 2026-04-21
sources: [papers/schonberger2016_colmap-sfm.md]
local_paper: papers/sfm-slam/he_2023_detector-free-sfm.pdf
url: https://arxiv.org/abs/2306.15669
code: https://github.com/zju3dv/DetectorFreeSfM
license_paper: arxiv-nonexclusive
license_code: Apache-2.0
status: draft
---

📄 [Full paper](../../papers/sfm-slam/he_2023_detector-free-sfm.pdf) · [arXiv](https://arxiv.org/abs/2306.15669) · [code](https://github.com/zju3dv/DetectorFreeSfM)

_Paper license: `arxiv-nonexclusive` · Code license: `Apache-2.0`_

## TL;DR
SfM framework that drops the `detect → describe → match` front-end in favor of semi-dense detector-free matchers (LoFTR-class), and solves the multi-view inconsistency those matchers produce via **match quantization** at the matcher's native coarse stride. A coarse SfM is reconstructed from quantized matches; an iterative refinement loop then alternates a transformer-based multi-view feature-track refiner with geometric BA and **track topology adjustment**. First place in Image Matching Challenge 2023; outperforms detector-based SfM (SP+SG+PixSfM) on ETH3D, IMC, and a new texture-poor dataset.

## Problem
Traditional SfM (COLMAP, Bundler) requires **repeatable keypoints** across ≥3 views as the first step. On texture-poor, repetitive, or low-contrast scenes (moon surface, underwater, untextured objects) SIFT/SuperPoint find almost no repeatable detections — the whole pipeline collapses. Detector-free matchers (LoFTR, AspanFormer, MatchFormer) produce strong *pairwise* correspondences on such scenes but are **multi-view-inconsistent**: matching `I_j` to `I_i` and `I_j` to `I_k` places keypoints at different sub-pixel locations in `I_j`, so feature tracks fragment. PatchFlow / PixSfM refine tracks post-hoc but (a) operate on detector-based inputs and (b) require full feature patches / cost maps in memory for feature-metric BA — prohibitive at detector-free match counts.

## Method
Two-stage coarse-to-fine pipeline:

**Stage 1 — Coarse SfM on quantized matches.** Run a detector-free matcher (LoFTR default) on image pairs. Quantize every matched location by `⌊x/r⌋·r` with `r = 8` (LoFTR's coarse stride). This grid-rounding **merges** pair-dependent sub-pixel matches into view-consistent "keypoints" and repairs track identity. Feed quantized keypoints into COLMAP's incremental mapping unchanged — pose accuracy is coarse but tractable.

**Stage 2 — Iterative refinement.** Alternate two sub-passes per iteration:

1. *Feature-track refinement*: for each track `T_j`, pick a reference view (middle-scale by current depth), extract `p×p` (p=15) patches centered on each keypoint across all views, feed through a 2-layer self/cross-attention transformer with S2DNet CNN backbone. Correlate reference feature with each query feature patch → per-query heatmap. Expectation = refined location; variance = per-view uncertainty. Repeat for a `w×w` (w=7) grid of reference locations, **select the reference location whose candidate track has minimum summed variance**. Output: refined `T_j*`.
2. *Geometry refinement*: plain geometric BA (minimize reprojection on `T_j*`, not feature-metric — this is why memory stays small) followed by **track topology adjustment**: project refined 3D points back into images, merge previously-unregistered 2D keypoints into tracks where reprojection falls below a looser threshold, complete broken tracks, filter outliers, re-triangulate. Alternate BA ↔ TA 5 times per iteration.

Trained on MegaDepth with a feature-track L2 loss between refined and ground-truth keypoint locations.

## Results
| Dataset | Method | AUC@10° / AUC@5cm |
|---|---|---|
| ETH3D | SP+SG+PixSfM (best detector-based) | 72.86 |
| ETH3D | Ours (LoFTR) | **79.53** |
| IMC 2021 | SP+SG+PixSfM | 70.47 |
| IMC 2021 | Ours (LoFTR) | **72.19** |
| Texture-Poor SfM (new, 17 scenes) | SP+SG+PixSfM | 24.55 |
| Texture-Poor SfM | Ours (LoFTR) | **45.43** |

3D triangulation on ETH3D: 80.38% @ 1cm accuracy vs 74.42% (LoFTR+PixSfM), 76.18% (SIFT+PixSfM). Large-scale Aachen (2000 images): peak BA memory 2.63 GB vs PixSfM 904.5 GB (**~340× less memory**). Refinement time 2319s vs PixSfM 2319s — roughly equivalent compute, dramatically lower memory.

Ablations (Table 3): match quantization `r=8` optimal; transformer gives +2.5 AUC@2cm over CNN-only features; reference-location search `w=7` gives +3.7%; track topology adjustment gives +4.6% on the strict 1cm threshold. Two refinement iterations is the compute/accuracy sweet spot.

## Why it matters
Detector-free matching has been the correspondence SOTA since LoFTR (2021) but was locked out of classical SfM because tracks were multi-view-inconsistent. This paper is the bridge — it lets any detector-free matcher drive a COLMAP-style mapper with higher accuracy *and* drastically lower BA memory than PixSfM, unlocking texture-poor SfM at scale. Together with [[pataki2025_mp-sfm]] (mono-depth-driven SfM on low-overlap captures), it closes two complementary failure modes of classical SfM: MP-SfM handles **low-overlap**, DetectorFreeSfM handles **low-texture**. The mechanisms are orthogonal and composable — flagged as a synthesis bet in [[feed-forward-structure-from-motion]] / [[gpu-native-sfm]].

## Pipeline contribution

- [[coarse-to-fine-detector-free-sfm-bridge_he2023]] — `scope: bridge`. Candidate thread: [[feed-forward-structure-from-motion]] Tier 2 + [[gpu-native-sfm]] op:general-purpose · stage: retypes [[feature-matching.task-head]] output into [[sfm.view-graph-construction]] input via match quantization · expected gain: unlocks detector-free matchers as SfM frontends on texture-poor scenes; ETH3D AUC@10° 79.53 vs 72.86 best detector-based; Texture-Poor 45.43 vs 24.55.
- [[multi-view-transformer-track-refinement_he2023]] — `scope: stage-swap` on new stage [[sfm.feature-track-refinement]]. Candidate thread: [[feed-forward-structure-from-motion]] Tier 2. Replaces PixSfM feature-metric BA / PatchFlow dense flow. Expected gain: 340× less BA memory than PixSfM on 2000-image Aachen; +2.5 AUC@2cm from multi-view attention vs CNN-only features.
- [[iterative-ba-plus-track-topology-adjustment_he2023]] — `scope: stage-split` on [[sfm.iterative-triangulation-ba]], introducing [[sfm.track-topology-adjustment]]. Candidate thread: [[feed-forward-structure-from-motion]] Tier 2 + [[gpu-native-sfm]] op:general-purpose · reusable independently in any incremental SfM that wants to recover tracks missed by the strict initial registration threshold. Expected gain: +4.6% on strict 1cm accuracy (Table 3 (4)).

All three ideas are bundle-locked via `co_requires:` — the DetectorFreeSfM system requires all three. Individual ideas are reusable against different downstream pipelines only if their own `co_requires:` chain is honored.

## Relation to prior work
- Extends [[colmap-incremental-sfm_schonberger2016]] — reuses COLMAP mapping math, changes the front-end + adds coarse-to-fine loop.
- Orthogonal to [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] — MP-SfM handles low-overlap via mono-depth priors; this paper handles low-texture via detector-free matching. Different failure modes, complementary solutions.
- Replaces [Lindenberger et al. 2021](lindenberger2021_pixsfm.md) PixSfM's feature-metric BA — track refinement via attention-transformed features + plain geometric BA is far more memory-efficient.
- Uses [Sun et al. 2021](sun2021_loftr.md) LoFTR as the default matcher; also tested with AspanFormer and MatchFormer.
- Contrasts with end-to-end pose-regression (DUSt3R/MASt3R family) — keeps classical BA, swaps only the front-end; bet on hybrid tier.

## Open questions / limitations
- **Speed at scale**: detector-free match counts are large; total mapping time (coarse SfM + refinement) is slower than classical detector-based pipelines on large-scale outdoor scenes. Limitation stated explicitly.
- **Matcher dependence**: quantization radius `r` is tied to matcher coarse stride. A matcher with a different stride needs `r` re-tuning.
- **My skepticism**: track-topology adjustment is reported to gain 4.6% but with fixed thresholds — cross-dataset robustness of the thresholds (especially the "looser" merge threshold) is under-ablated. Also unclear how the w×w reference-location search interacts with symmetric / repetitive patterns — the paper mentions minimum-variance selection can pick the wrong reference on symmetric content.
- **Not tested**: pairing detector-free front-end with MP-SfM's mono-depth-based registration (the natural texture-poor + low-overlap combo) or with InstantSfM's GPU-native BA (the natural memory/speed combo).

## Code & license
Apache-2.0 on code, standard arxiv-nonexclusive on paper. No commercial-use blockers. Depends on LoFTR (Apache-2.0) and COLMAP (BSD). Fully commercial-OK.

## References added to the wiki
- [[coarse-to-fine-detector-free-sfm-bridge_he2023]] (new idea)
- [[multi-view-transformer-track-refinement_he2023]] (new idea)
- [[iterative-ba-plus-track-topology-adjustment_he2023]] (new idea)
- [[sfm.feature-track-refinement]] (new stage)
- [[sfm.track-topology-adjustment]] (new stage)
- [[sun2021_loftr|LoFTR]] (new stub)
- [[lindenberger2021_pixsfm|PixSfM]] (new stub)
