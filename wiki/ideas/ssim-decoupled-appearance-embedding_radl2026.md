---
title: SSIM-decoupled appearance embedding (luminance-only compensation)
type: idea
source_paper: wiki/papers/radl2026_confidence-mesh-3dgs.md
also_in: []

# pipeline shape
scope: stage-swap
stages: [radiance-fields.appearance-compensation]
collapses: []
splits_into: []
rewrites: {}

# type contracts
inputs: [rendered-image, per-image-appearance-embedding, gt-image]
outputs: [transformed-rendered-image-for-L1, untouched-rendered-for-contrast-structure, total-loss]
assumptions: [static-scene, appearance-embedding-per-training-image, training-time-only]
requires_upstream_property: [per-image-appearance-embedding-available]
requires_downstream_property: [accepts-split-L1-and-structural-loss]
learned_params: [appearance-embedding-vector-per-image, appearance-cnn-weights]
failure_modes: [embedding-capacity-too-small-for-extreme-exposure-shifts]

# composition graph
requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: [decoupled-appearance-2d-transform_lin2024]
contradicts: []

tags: [3dgs, appearance-embedding, ssim, vastgaussian-refinement]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

VastGaussian splits the loss so L1 uses the per-image CNN-transformed render and D-SSIM uses the untransformed render. CoMe refines this: it decomposes D-SSIM into its three factors — luminance `l`, contrast `c`, structure `s` — and lets the appearance embedding compensate **only for luminance**, while `c` and `s` use the original rendered image. This prevents the appearance CNN from masking geometric errors by adjusting structure/contrast to match GT.

## Why it wins

Causal claim: appearance embeddings trained against full D-SSIM can "cheat" by fitting structural differences caused by geometry errors; restricting compensation to luminance forces those geometric errors to appear in the structural residual where the radiance loss can correct them. The paper's Table 3 ablation isolates the SSIM-decoupling contribution (~+0.01 F1 on T&T; small but consistent).

## Preconditions & compatibility

Requires VastGaussian's per-image appearance embedding + CNN transformation-map infrastructure (refinement, not replacement). Compatible with any 3DGS pipeline that uses L1 + D-SSIM as its photometric loss.

## Open questions

- Does the same l/c/s split help non-D-SSIM photometric losses (e.g. LPIPS)?
- Compatibility with bilateral-grid-based appearance models.
