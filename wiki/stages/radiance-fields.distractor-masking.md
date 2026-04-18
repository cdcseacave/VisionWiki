---
title: Radiance-fields distractor masking
type: stage
slug: radiance-fields.distractor-masking
consumes: [rendered-rgb, gt-rgb, residuals, pixel-segmentation]
produces: [per-pixel-distractor-mask]
invariants: [static-content-preserved]
provides_properties: [photometric-loss-robust-to-dynamic-objects]
requires_upstream_properties: [residual-signal-or-segmentation-available]
data_regime: [dynamic-or-in-the-wild-capture]
tags: [3dgs, distractor, in-the-wild, adaptive-threshold]
created: 2026-04-18
updated: 2026-04-18
---

Masks out dynamic / transient content from the photometric loss. Fixed-threshold methods (RobustNeRF) or semantic-category methods (NeRF-HuGS) are simpler baselines; adaptive residual-based thresholds (DroneSplat) handle variable distractor levels without per-scene tuning. Example fillers: [[adaptive-local-global-distractor-masking_tang2025]].
