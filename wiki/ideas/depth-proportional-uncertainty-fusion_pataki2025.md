---
title: Depth-proportional uncertainty fusion for mono-depth priors
type: idea
source_paper: wiki/papers/pataki2025_mp-sfm.md
also_in: []

scope: drop-in
stages: [sfm.depth-constrained-ba, mvs.depth-refinement]
collapses: []
splits_into: []
rewrites: {}

inputs: [mono-depth-estimate, mono-depth-uncertainty-from-model, ground-truth-depth-on-calibration-split]
outputs: [recalibrated-per-pixel-depth-uncertainty]
assumptions: [mono-depth-model-outputs-per-pixel-uncertainty-or-we-synthesize-it]
requires_upstream_property: [mono-depth-estimator-with-any-uncertainty-signal-or-we-fabricate-proportional]
requires_downstream_property: [downstream-uses-per-pixel-depth-covariance]
learned_params: [single-scalar-calibration-factor-per-estimator]
failure_modes: [distribution-shift-from-calibration-to-test-data-degrades-fusion, clipping-min-hides-true-certainty-on-close-surfaces]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mono-depth, uncertainty, calibration, mp-sfm]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Off-the-shelf monocular depth models (Metric3Dv2, DepthPro, DepthAnything-v2) either emit systematically over-confident per-pixel uncertainties or emit none at all. The recipe:

1. **Depth-proportional uncertainty**: $\sigma_{\text{prop}}(u,v) = \alpha \cdot D(u,v)$. Error in mono-depth scales with depth; this captures that lower bound.
2. **Model-predicted uncertainty**: $\sigma_{\text{pred}}(u,v)$ from the estimator (or a flip-consistency proxy for estimators without one).
3. **Per-pixel fusion**: $\sigma(u,v) = \max\left( \sigma_{\text{prop}}(u,v), \sigma_{\text{pred}}(u,v) \right)$ — take the conservative estimate at every pixel.
4. **Clip min**: floor $\sigma$ at 2 cm to reject over-confident near-surface predictions.
5. **Calibration**: tune $\alpha$ (and an overall scale on $\sigma_{\text{pred}}$) once on a held-out calibration split (e.g., ETH3D training) by maximizing AUC of the prior-sensitivity plot (RMSE vs. recall, filtering by uncertainty).

For MASt3R depth specifically, step (2) alone is already well-calibrated and step (1)/(3) can be skipped.

## Why it wins

Fig. 7 in the paper's Appendix C — calibration plot: raw Metric3Dv2 uncertainties lie far below the y=x line (over-confident); the combined (fused + clipped) uncertainty hugs y=x. Table 4 confirms downstream SfM AUC gains: row 2 (no depth uncertainty) vs row 1 (with): SMERF minimal-overlap AUC@5° 37.7 → 40.1.

The causal story: downstream BA / depth refinement (inverse-variance weighting) collapses when priors claim certainty they don't have, since the optimizer trusts wrong predictions. Proper calibration restores the intended robustness-to-noise behavior.

## Preconditions & compatibility

- Needs a modest calibration split (tens of scenes with GT depth). Absent that, depth-proportional-only ($\alpha$ from a reasonable default) still beats raw model outputs.
- Drop-in for any pipeline consuming mono-depth with uncertainty: classical SfM (MP-SfM, [[depth-constrained-jacobian_zhong2026]]), 3DGS depth supervision ([[median-depth-relative-loss_kim2025]], [[va-gs-four-loss-stack_li2025]]), MVS depth refinement, depth completion.
- Cannot restore information the underlying mono-depth doesn't have — if the estimator is systematically wrong on vegetation/specular surfaces, calibration cannot fix that.

## Trade-offs vs. the decomposed pipeline

Not applicable — `drop-in` scope.

## Open questions

- Does the calibration transfer zero-shot across datasets (ETH3D → SMERF → outdoor drone)? The paper calibrates once on ETH3D training and uses the same $\alpha$ everywhere, with good results, but systematic cross-domain evaluation is missing.
- Is the $\max$ fusion optimal, or does a learned per-pixel gate (input: depth, predicted-uncertainty) do better?
- For estimators without uncertainty heads, is flip-consistency the best proxy, or do test-time ensembles / dropout estimates beat it?
