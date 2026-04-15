---
title: Semi-Global Matching (SGM)
type: method
tags: [stereo, mvs, classical, dense-matching]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/chebbi2025_multiview-dense-matching.md]
status: stub
---

## TL;DR
Classical semi-global stereo matching (Hirschmüller 2005/2008): approximates a 2D MRF with 1D dynamic-programming passes along multiple directions. Still widely used as a depth-regularization backend in modern learned MVS (e.g. MS-AFF).

## Relation
- Baseline for [[multi-view-stereo]]
- Used as post-regularization in learned pipelines like [Chebbi et al. 2025](../papers/chebbi2025_multiview-dense-matching.md)
