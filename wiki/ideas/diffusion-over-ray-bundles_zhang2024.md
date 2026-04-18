---
title: RayDiffusion — denoising diffusion over ray bundles for pose uncertainty
type: idea
source_paper: wiki/papers/zhang2024_cameras-as-rays.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.uncertainty-aware-pose-output]
inputs: [noisy-ray-bundle, dinov2-features]
outputs: [denoised-ray-bundle-samples, multi-modal-pose-distribution]
assumptions: [diffusion-training-budget, inference-time-sampling-acceptable]
requires_upstream_property: [plucker-ray-bundle-regressor]
requires_downstream_property: [consumer-uses-sample-diversity-as-uncertainty]
learned_params: [diffusion-denoiser-weights]
failure_modes: [sampling-cost-vs-single-shot-regression]

requires: [plucker-ray-bundle-camera_zhang2024]
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, diffusion, ray-bundle, uncertainty]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Diffusion posterior over ray bundles: start from Gaussian-noise rays, iteratively denoise conditioned on DINOv2 features. Sample diversity captures symmetry / partial-observation ambiguities that point-regression cannot express.

## Why it wins

Multi-modal pose distributions under ambiguous inputs — a property no deterministic regression head provides. Natural when downstream pipelines (planning, BA init) benefit from uncertainty-aware inputs.

## Open questions

- How many diffusion steps are needed at deployment? Compute vs. uncertainty trade unexplored.
