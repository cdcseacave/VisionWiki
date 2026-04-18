---
title: Feed-forward SfM uncertainty-aware pose output
type: stage
slug: feed-forward-sfm.uncertainty-aware-pose-output
consumes: [image-features, noisy-pose-or-ray-bundle]
produces: [multi-modal-pose-distribution]
invariants: [sample-diversity-reflects-true-ambiguity]
provides_properties: [multi-modal-output]
requires_upstream_properties: [diffusion-training-budget]
data_regime: [ambiguous-inputs-symmetries]
tags: [raydiffusion, diffusion-sfm, uncertainty]
created: 2026-04-18
updated: 2026-04-18
---

Diffusion-based pose output expresses multi-modal ambiguity via sample diversity. RayDiffusion and DiffusionSfM are the canonical fillers. Example fillers: [[diffusion-over-ray-bundles_zhang2024]], [[ray-origin-endpoint-diffusion_zhao2025]].
