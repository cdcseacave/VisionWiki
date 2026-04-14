---
title: GLOMAP
type: method
tags: [structure-from-motion, global-sfm, pose-estimation]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/pan2024_glomap.md]
status: draft
---

## What it is

GLOMAP is a **global structure-from-motion** pipeline that reconstructs camera poses and sparse geometry from a collection of views in a single non-incremental optimization. Positioned as a faster, more robust alternative to the incremental [[colmap|COLMAP]] pipeline for large image collections.

## How it works

- Operates on a view graph of pairwise relative poses from feature matches.
- Jointly solves rotation averaging and global translation estimation, followed by global bundle adjustment.
- Avoids the drift and speed penalties of incremental SfM by skipping per-image registration.

## Relation to prior work

- Successor / alternative to [[colmap|COLMAP]]'s incremental pipeline.
- Frequently cited as a baseline alongside COLMAP in 2025–2026 SfM papers (see [[gpu-native-sfm]]).

## Strengths
- Faster on large collections; better parallelization.
- Less sensitive to initialization order.

## Limitations
- Less robust on very sparse / low-overlap captures than incremental methods.
- Quality depends heavily on the pairwise view-graph stage.

## Key references
- [Pan et al. 2024](../papers/pan2024_glomap.md) · [pdf](../../papers/sfm-slam/pan_2024_glomap.pdf) — canonical GLOMAP paper, ECCV 2024.
