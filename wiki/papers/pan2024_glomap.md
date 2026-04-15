---
title: "GLOMAP: Global Structure-from-Motion Revisited"
type: paper
tags: [sfm, global-sfm, bundle-adjustment, colmap]
created: 2026-04-14
updated: 2026-04-15
sources: []
local_paper: papers/sfm-slam/pan_2024_glomap.pdf
url: https://arxiv.org/abs/2407.20219
code: https://github.com/colmap/glomap
license_code: BSD-3-Clause
license_paper: CC-BY-4.0
status: stable
---

📄 [Full paper](../../papers/sfm-slam/pan_2024_glomap.pdf) · [arXiv](https://arxiv.org/abs/2407.20219) · [GitHub](https://github.com/colmap/glomap) · [code](https://github.com/colmap/glomap)

_Code license: `BSD-3-Clause`_

## TL;DR

Pan, Bárath, Pollefeys & Schönberger (ECCV 2024) revisit **global [[structure-from-motion]]** and propose GLOMAP, a pipeline that matches [[colmap|COLMAP]]-level accuracy and robustness while running **orders of magnitude faster**. Released as open source under the COLMAP umbrella.

## Problem

Incremental SfM (COLMAP) is the de facto accuracy/robustness leader but scales poorly: each image is registered and BA'd sequentially. Global SfM is faster and parallelizable but historically brittle — small errors in rotation averaging or global translation estimation poison the downstream [[bundle-adjustment|bundle adjustment]].

## Method

- **Joint global positioning**: replaces the classical translation-averaging step with a direct joint optimization of camera positions *and* 3D points, using a robust loss over the view-graph.
- **Improved rotation averaging** with outlier-aware weighting.
- **Single final BA** over the globally initialized reconstruction — no iterative registration loop.
- Matches COLMAP's input format (features, two-view geometries, view graph), making it a drop-in backend swap.

## Results

- On 1DSfM, ETH3D, IMC PhotoTourism, and MIP-360 collections: pose errors within noise of COLMAP; often better on large sequences.
- **Speedup**: reported up to 2–3 orders of magnitude faster than incremental COLMAP on large collections.
- Robust on unordered internet photo collections — historically the weak point of global methods.

## Why it matters

Validates that global SfM, when the joint-position stage is formulated correctly, is no longer an accuracy-for-speed trade but a strict Pareto improvement on incremental SfM for most workloads. Downstream implications for [[gpu-native-sfm]] and any system that uses SfM as a preprocessing step (NeRF, 3DGS, MVS).

## Pipeline contribution

- **Joint global positioning (camera centers + 3D points in one solve)** — replaces translation-averaging → BA with a single robust joint optimization. candidate thread: [[gpu-native-sfm]] · stage: *global-SfM initialization* · replaces/augments: *classical translation averaging* · expected gain: matches COLMAP accuracy at 2–3 orders of magnitude faster on large collections; raises the classical baseline that feed-forward methods must beat.
- **Outlier-aware rotation averaging** — robust weighting on the rotation-only subproblem. candidate thread: [[gpu-native-sfm]] · stage: *rotation averaging* · replaces/augments: *L2 rotation averaging* · expected gain: robustness on noisy view graphs.
- **Drop-in COLMAP backend compatibility** — same frontend, same file format. candidate thread: [[gpu-native-sfm]] · stage: *systems integration* · expected gain: trivial adoption in downstream pipelines.
- **Role in the wiki**: GLOMAP is the **current accuracy/speed Pareto point** for global SfM on CPU; InstantSfM ([[zhong2026_instantsfm]]) extends this to GPU, 12× faster than GLOMAP. In the [[gpu-native-sfm]] lineage it is the 2024 step between COLMAP and InstantSfM.

## Relation to prior work

- Builds on [[colmap|COLMAP]]'s feature/matching frontend.
- Supersedes prior global-SfM systems (Theia, OpenMVG global) on robustness.
- Contextualizes recent feed-forward SfM ([[feed-forward-structure-from-motion]]): GLOMAP raises the classical baseline these methods must beat.

## Open questions / limitations

- Behavior on very sparse / low-overlap captures (classical weak spot of global SfM) not fully characterized.
- Still requires a good feature frontend; failure on degenerate or textureless scenes inherits from matching.

## References added to the wiki

- [[glomap]] (stub expanded with sources).
