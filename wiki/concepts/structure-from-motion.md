---
title: Structure from Motion
type: concept
tags: [sfm, 3d-reconstruction, pose-estimation]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/schonberger2016_colmap-sfm.md, wiki/papers/pan2024_glomap.md]
status: draft
---

## What it is

Structure from Motion (SfM) is the joint estimation of **camera poses** and **sparse 3D structure** from an unordered set of 2D images. It is the classical entry point to multi-view geometry and the upstream step for [[multi-view-stereo]], meshing, and radiance-field fitting.

## Canonical pipeline

1. **Feature detection & matching** — keypoints ([[feature-matching]]) across images.
2. **Geometric verification** — fundamental/essential matrix, RANSAC.
3. **Initialization** — two-view or multi-view seed reconstruction.
4. **Registration** — add cameras via PnP.
5. **Triangulation** — recover 3D points.
6. **[[bundle-adjustment|Bundle adjustment]]** — joint refinement of poses + points.

## Paradigms

| Approach | Representative | Characteristics |
|----------|----------------|-----------------|
| Incremental | [[colmap|COLMAP]] | Robust, slow, drift-prone on large sets |
| Global | [[glomap|GLOMAP]] | Fast, parallel, needs good view graph |
| Hierarchical | — | Divide-and-merge |
| Feed-forward / learned | [[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]] | End-to-end neural; no explicit BA loop (see [[feed-forward-structure-from-motion]]) |
| GPU-native | See [[gpu-native-sfm]] | Batched classical SfM on GPU |

## Open questions
- Where is the crossover between classical optimization and neural feed-forward pipelines in accuracy / scale?
- Can learned methods produce calibration-quality poses or only "good enough" for downstream tasks?

## Key references
- [Schönberger & Frahm 2016 (COLMAP SfM)](../papers/schonberger2016_colmap-sfm.md) · [pdf](../../papers/sfm-slam/schonberger_2016_colmap-sfm.pdf) — CVPR 2016; canonical incremental SfM pipeline.
- [Pan et al. 2024 (GLOMAP)](../papers/pan2024_glomap.md) · [pdf](../../papers/sfm-slam/pan_2024_glomap.pdf) — global SfM, orders of magnitude faster than COLMAP.
- Hartley & Zisserman, *Multiple View Geometry in Computer Vision* (2003) — the textbook reference.
