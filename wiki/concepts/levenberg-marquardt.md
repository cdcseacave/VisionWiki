---
title: Levenberg–Marquardt
type: concept
tags: [optimization, nonlinear-least-squares, bundle-adjustment]
created: 2026-04-15
updated: 2026-04-15
sources: [papers/li2025_megasam.md, papers/zhong2026_instantsfm.md]
status: stub
---

## What it is
Nonlinear least-squares optimization algorithm that interpolates between Gauss–Newton and gradient descent via a damping parameter. The workhorse of [[bundle-adjustment]] and most photogrammetric refinement.

## Where it appears
- Bundle adjustment backends: Ceres, g2o, GTSAM
- GPU-native SFM: [[zhong2026_instantsfm|InstantSFM]] uses per-iteration active-set LM
- Hybrid SFM+depth: [[li2025_megasam|MegaSaM]]
