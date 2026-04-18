---
title: Per-camera Jacobian-based whitening preconditioner (CamP)
type: idea
source_paper: wiki/papers/park2023_camp.md
also_in: []

scope: drop-in
stages: [radiance-fields.joint-pose-radiance-optimization]
inputs: [camera-parameters, proxy-projection-jacobian]
outputs: [preconditioned-parameterization, improved-convergence]
assumptions: [joint-pose-radiance-optimization-attempted, cameras-near-initial-sfm-accuracy]
requires_upstream_property: [sfm-initial-poses-available]
requires_downstream_property: [renderer-accepts-preconditioned-cameras]
learned_params: []
failure_modes: [preconditioner-computed-once-from-init-no-dynamic-update, local-minima-still-possible]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [nerf, 3dgs, pose-optimization, preconditioning, camp]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Compute the proxy Jacobian `J` of projection residuals w.r.t. camera parameters; take the Cholesky factor `P` of `J^T J`. Reparameterize: optimize `φ` where `θ = P φ + θ_0`. The preconditioner whitens the ill-conditioned camera parameters — SO(3) angles, translation, focal length all have different curvatures; `P` normalizes them.

## Why it wins

67% RMSE reduction vs. non-optimizing baseline; 29% over prior camera-optimizing NeRF. Recovers from ~0.4° orientation error to near-COLMAP accuracy. Stacked on Zip-NeRF it's the NeRF-family ceiling on noisy-pose datasets.

## Preconditions & compatibility

The preconditioner is computed once from initial SfM points — dynamic update during training would likely help (paper's own open question). **Representation-agnostic**: depends only on camera-projection Jacobian. Bet #003 (port CamP to 3DGS) is the canonical cross-representation bet.

## Open questions

- Why does no 3DGS paper replicate this trick despite it being representation-agnostic?
- Using SfM feature points (rather than uniform samples) for the Jacobian may improve conditioning.
