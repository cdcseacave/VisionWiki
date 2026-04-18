---
title: Ray origin + endpoint diffusion for unified N-view SfM (DiffusionSfM)
type: idea
source_paper: wiki/papers/zhao2025_diffusionsfm.md
also_in: []

scope: topology-rewrite
stages: [feed-forward-sfm.camera-output, feed-forward-sfm.geometry-output]
collapses: []
splits_into: []
rewrites: {replaces: [feed-forward-sfm.pose-regression, feed-forward-sfm.pointmap-prediction], introduces: [feed-forward-sfm.ray-origin-endpoint-diffusion]}

inputs: [n-view-images, dinov2-features]
outputs: [per-pixel-ray-origin-endpoint-fields, camera-extrinsics-and-intrinsics-invertible]
assumptions: [diffusion-training-budget, per-pixel-output-resolution]
requires_upstream_property: [dinov2-backbone-features]
requires_downstream_property: [downstream-inverts-ray-origins-to-cameras]
learned_params: [diffusion-denoiser-weights]
failure_modes: [unbounded-scene-depth-diffusion-stability]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: [plucker-ray-bundle-camera_zhang2024]
contradicts: []

tags: [feed-forward-sfm, diffusion, ray-origin-endpoint, uncertainty]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Each pixel gets `(O_{ij}, E_{ij})` — ray origin + endpoint in world frame — as a per-pixel prediction. Ray origins can be inverted to camera extrinsics/intrinsics in closed form. Denoising diffusion reverses noise over the ray-origin+endpoint fields conditioned on DINOv2 features. **Homogeneous-coordinate normalization** bounds unbounded geometry for stable diffusion; **GT-mask conditioning** informs about missing depth.

## Why it wins

Single-pass N-view reasoning without pairwise → global-alignment cascade (DUSt3R's O(N²) pattern). Built-in uncertainty via sample diversity. Eliminates the separate pose-regression + pointmap + alignment stages of DUSt3R-family methods.

## Pipeline-shape implications

Topology-rewrite: replaces DUSt3R-style pointmap + pose stages with a unified ray-origin+endpoint diffusion. Threads adopting this change their SfM pipeline's shape.

## Open questions

- Unbounded scene depths stress the diffusion process; homogeneous normalization is an engineering fix, not a theoretical one.
