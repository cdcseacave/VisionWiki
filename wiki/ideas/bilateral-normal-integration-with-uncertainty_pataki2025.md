---
title: Bilateral normal integration with uncertainty-weighted BA residual
type: idea
source_paper: wiki/papers/pataki2025_mp-sfm.md
also_in: []

scope: stage-swap
stages: [sfm.depth-constrained-ba, mvs.depth-refinement]
collapses: []
splits_into: []
rewrites: {}

inputs: [prior-mono-depth, prior-mono-normals, per-pixel-depth-and-normal-uncertainty]
outputs: [refined-depth-map-consistent-with-normals-under-propagated-uncertainty]
assumptions: [surface-smoothness-locally, normal-prior-roughly-consistent-with-depth-prior]
requires_upstream_property: [monocular-depth-and-surface-normal-both-available-with-uncertainty]
requires_downstream_property: [ba-solver-accepts-per-pixel-residuals-with-propagated-covariance]
learned_params: []
failure_modes: [normal-depth-contradiction-on-specular-or-translucent-surfaces, cost-dominance-without-robust-loss]

requires: []
unlocks: []
co_requires: [depth-proportional-uncertainty-fusion_pataki2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [bundle-adjustment, normal-integration, mono-prior, uncertainty-propagation]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Extends Cao et al. 2022 bilateral normal integration by propagating per-pixel normal uncertainty into the integration residual. The residual at pixel $(u,v)$ is $\mathbf{r}_i = \mathbf{N}_i(u,v) - \Delta D^*_i(u,v) \in \mathbb{R}^4$, coupling predicted surface normals $\mathbf{N}_i$ with the discretized one-sided depth gradients $\partial^\pm_{u/v} D^*$ of the refined depth $D^*$ (the four-entry residual captures the four discretization directions).

Uncertainty derivation:
1. Predicted spherical-coordinate normal uncertainty $\Sigma_{\theta,\varphi}$ → Cartesian via Jacobian $J_{xyz}$: $\Sigma_{xyz} = J_{xyz}\Sigma_{\theta,\varphi}J_{xyz}^\top$.
2. Optional flip-consistency augmentation: max of predicted and flip-derived normal variances per component.
3. Approximate residual covariance $\sigma_{N^\pm_{u/v}}^2 \approx \sigma_{N_{u/v}}^2$ via linearization (the depth term's contribution becomes small under mild conditions).

Robustified with truncated-L2 losses `ρ_prior` / `ρ_int` (the depth-prior side uses a smooth-L1 / Cauchy depending on phase). Combined with the depth-prior term $\rho_{\text{prior}}(\|D^*(u,v) - D(u,v)\|_{\Sigma_{D}}^2)$ into the $C_{int}$ objective.

## Why it wins

Table 5 "no depth refinement" row — removing the normal-integration cost leaves the prior depth (high-variance, surface-texture-dominated) as the only structure signal. MP-SfM numbers show material degradation on ETH3D minimal-overlap AUC@1° (23.6 → 27.3 when refinement restored). On Table 5 rows "no depth reg" the depth-only regularizer without normals also underperforms, isolating the normal-integration term.

Mechanism: normals encode surface tangent orientation, which constrains depth gradients to match observed surface geometry even in textureless regions where multi-view reprojection gives no signal. Propagated covariance lets the solver discount unreliable normals (e.g., on ground-plane shadows, thin structures) instead of fighting them.

## Preconditions & compatibility

- Requires a normal estimator in addition to depth. Metric3Dv2 provides both; DSINE is an alternative normal-only replacement. Without both, this idea cannot be instantiated.
- Requires calibrated uncertainties — hence `co_requires: [depth-proportional-uncertainty-fusion_pataki2025]` when normals come from estimators with uncalibrated uncertainty.
- Candidate port into any joint-depth-pose refinement stage: 3DGS pose refinement ([[radiance-fields.joint-pose-radiance-optimization]]), MVS post-refinement ([[mvs.depth-refinement]]), feed-forward SfM with test-time refinement heads.

## Trade-offs vs. the decomposed pipeline

Not applicable — `stage-swap` sub-mechanism of [[sfm.depth-constrained-ba]].

## Open questions

- On outdoor / vegetation / specular surfaces, how much does over-confident normal prediction erode the benefit? Paper acknowledges limitations but doesn't quantify.
- Could a learned gate modulate the integration cost per region (e.g., based on a confidence map) and beat the static robust-loss schedule?
- The full residual propagation assumes small-perturbation linearization — does this break on scenes with large depth range where the one-sided gradient approximation is poor?
