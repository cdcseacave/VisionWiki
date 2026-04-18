---
title: Adaptive local-global distractor masking for aerial 3DGS
type: idea
source_paper: wiki/papers/tang2025_dronesplat.md
also_in: []

scope: drop-in
stages: [radiance-fields.distractor-masking]
inputs: [rendered-rgb, gt-rgb, residual-statistics, pixel-level-segmentation]
outputs: [per-pixel-distractor-mask, masked-photometric-loss]
assumptions: [dynamic-distractors-present, residuals-informative-of-distractors]
requires_upstream_property: [residual-computation-differentiable]
requires_downstream_property: [photometric-loss-accepts-per-pixel-mask]
learned_params: [threshold-adaptation-parameters]
failure_modes: [small-distractor-dataset-only-24-sequences-generalization-untested]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, aerial, distractor-masking, adaptive]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Residual-based statistical distractor detection at two scales (local pixel + global image) combined with pixel-level segmentation. Thresholds **adapt in real time** from the residual distribution rather than being fixed hyperparameters. Mask is applied to the photometric loss so dynamic-object pixels are excluded from supervision.

## Why it wins

Best PSNR/SSIM/LPIPS on DroneSplat + NeRF On-the-go at all distractor levels (low/med/high) without per-scene tuning. Fixed-threshold methods (RobustNeRF) and semantic-category methods (NeRF-HuGS) break across distractor regimes; adaptive thresholds cover the range.

## Preconditions & compatibility

Natural composition with [[decoupled-appearance-2d-transform_lin2024]]: VastGaussian handles appearance drift, DroneSplat handles dynamic distractors — orthogonal failure modes, should compose cleanly (no paper does the composition).

## Open questions

- DroneSplat dataset is small (24 sequences); broader generalization untested.
