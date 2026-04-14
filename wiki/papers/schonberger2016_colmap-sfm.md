---
title: "Structure-from-Motion Revisited"
type: paper
tags: [sfm, incremental-sfm, bundle-adjustment, colmap]
created: 2026-04-14
updated: 2026-04-14
sources: []
local_paper: papers/sfm-slam/schonberger_2016_colmap-sfm.pdf
url: https://openaccess.thecvf.com/content_cvpr_2016/papers/Schonberger_Structure-From-Motion_Revisited_CVPR_2016_paper.pdf
status: stable
---

📄 [Full paper](../../papers/sfm-slam/schonberger_2016_colmap-sfm.pdf) · [CVPR 2016 PDF](https://openaccess.thecvf.com/content_cvpr_2016/papers/Schonberger_Structure-From-Motion_Revisited_CVPR_2016_paper.pdf) · [GitHub](https://github.com/colmap/colmap)

## TL;DR

Schönberger & Frahm (CVPR 2016) present **COLMAP**'s incremental [[structure-from-motion]] pipeline — a meticulously re-engineered take on every stage of the classical SfM loop (feature verification, scene graph, next-best-view, triangulation, BA). Becomes the default open-source SfM for the next decade.

## Problem

By 2016, incremental SfM was mature in principle but fragile in practice: reconstructions on large internet photo collections routinely failed or drifted. Prior systems (Bundler, VisualSFM) each had isolated strengths; none was simultaneously robust, accurate, scalable, and easy to deploy.

## Method

A re-audit of the incremental SfM pipeline with targeted improvements at every stage:
- **Geometric verification**: augmented with a multi-model RANSAC (homography vs. epipolar) to catch panorama/planar-degenerate matches.
- **Scene graph augmentation**: propagates cross-view consistency before initialization.
- **Next-best-view selection**: picks the next camera to register based on expected reconstruction gain (not raw match count).
- **Robust, iterative triangulation**: re-triangulates as BA refines poses, recovering tracks lost to initial noise.
- **Iterative bundle adjustment + re-triangulation loop** until convergence.
- Production-grade open-source implementation with vocab-tree matching, GPU SIFT, and dense MVS stage (the companion [Schönberger 2016 MVS paper](schonberger2016_colmap-mvs.md)).

## Results

- Substantially higher completeness and accuracy than Bundler / VisualSFM on internet photo collections (Rome, Dubrovnik) and controlled datasets.
- Robust where prior systems failed silently.
- Scales to tens of thousands of images with vocab-tree matching.

## Why it matters

COLMAP *is* the de facto SfM baseline. Nearly every 2017–2025 reconstruction paper — NeRF, 3DGS, MVS, feed-forward 3D — reports "ground-truth poses from COLMAP." Replacing COLMAP is an explicit goal of recent work: [[pan2024_glomap|GLOMAP]] (global), [[zhong2026_instantsfm|InstantSfM]] (GPU), [[dust3r|DUSt3R]]/[[mast3r|MASt3R]]/[[vggt|VGGT]] (feed-forward).

## Relation to prior work

- Builds on Bundler (Snavely 2006) and VisualSFM (Wu 2011); supersedes both.
- Incremental counterpart to the global [[pan2024_glomap|GLOMAP]] released under the same COLMAP umbrella in 2024.
- Companion paper: [Schönberger 2016 MVS](schonberger2016_colmap-mvs.md) adds dense reconstruction.

## Open questions / limitations

- Sequential incremental registration is slow; does not parallelize well.
- Drift on long trajectories without loop closure.
- SIFT frontend is not robust to drastic appearance / viewpoint changes (addressed in 2020+ by learned matchers).

## References added to the wiki

- [[colmap]] method page — canonical SfM citation added.
- [[structure-from-motion]] concept — `[!needs-source]` resolved.
