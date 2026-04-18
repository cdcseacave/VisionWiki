---
title: Per-primitive color + normal variance losses
type: idea
source_paper: wiki/papers/radl2026_confidence-mesh-3dgs.md
also_in: []

# pipeline shape
scope: drop-in
stages: [radiance-fields.regularization]
collapses: []
splits_into: []
rewrites: {}

# type contracts
inputs: [3d-gaussians, per-view-rendered-color, per-view-rendered-normal]
outputs: [regularization-loss-terms]
assumptions: [static-scene, posed-input, surface-adherent-target]
requires_upstream_property: [multi-view-rendering-available, per-primitive-attribution]
requires_downstream_property: [tolerates-regularization-gradient]
learned_params: []
failure_modes: [over-regularizes-thin-structure, penalizes-legitimate-specular-variance]

# composition graph
requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, regularization, variance-loss, surface-adherence]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

For each Gaussian visible across ≥2 views, accumulate the rendered color contribution and the rendered normal contribution per view. The variance of these per-Gaussian contributions across views is penalized: `L_color-var = Σ_i Var_v(c_i^v)`, `L_normal-var = Σ_i Var_v(n_i^v)`. Primitives that are view-inconsistent (off-surface floaters or noisy Gaussians) have high variance; surface-adherent primitives have low variance.

## Why it wins

Ablation Table 3: removing variance losses drops T&T F1 by ~0.03. The variance term specifically kills spurious geometry (floaters that look fine from one view but render badly from another). Complementary to the confidence loss — confidence balances losses, variance constrains the primitives themselves.

## Preconditions & compatibility

Needs a rasterizer that exposes per-primitive-per-view contributions (gsplat's feature-channel trick suffices — see [[nerfstudio]] for the gsplat escape hatch). In a forward-only rasterizer without this export, implementation requires a CUDA kernel change. Composes naturally with the confidence loss (they regularize different axes).

## Open questions

- How to handle legitimate view-dependent variance (specular). The confidence loss partly absorbs this; variance alone would be too aggressive.
- Cost of the variance accumulation at 1M+ primitives across 100+ views.
