---
title: Decoupled appearance modeling via 2D transformation map
type: idea
source_paper: wiki/papers/lin2024_vastgaussian.md
also_in: []

scope: stage-swap
stages: [radiance-fields.appearance-compensation]
inputs: [rendered-image, gt-image, per-image-appearance-embedding]
outputs: [transformed-render-for-L1, untouched-render-for-D-SSIM, total-loss]
assumptions: [pixel-wise-transform-suffices, training-time-only, per-image-embedding-learnable]
requires_upstream_property: [per-image-learnable-embedding-available]
requires_downstream_property: [accepts-split-L1-D-SSIM-loss]
learned_params: [per-image-appearance-embedding, small-cnn-weights]
failure_modes: [appearance-leaks-into-geometry-via-D-SSIM-contrast-and-structure, strong-specular-drift-unhandled]

requires: []
unlocks: [ssim-decoupled-appearance-embedding_radl2026]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, appearance-embedding, in-the-wild, rasterization-compatible]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: [come-integration-nerfstudio]
---

## Mechanism

Rendered image is downsampled 32×, concatenated with a per-image appearance embedding `ℓ_i` (length 64), passed through a small CNN (4 upsampling blocks + pixel-shuffle + 3×3 conv) to produce a pixel-wise transformation map `M_i`. Loss is split: L1 uses the transformed render, D-SSIM uses the untransformed render. At inference `M_i` and `ℓ_i` are discarded — no runtime overhead. The D-SSIM on the untransformed render is supposed to force Gaussians to learn consistent *structure* while the embedding absorbs *appearance*.

## Why it wins

Ablation Tab. 3: w/o decoupled appearance module → PSNR 26.81 → 25.08, floaters return. The 2D-transform-target approach is the **rasterization-native** answer to GLO embeddings: NeRF-W's MLP-conditioned radiance is incompatible with splatting's pre-computed SH; moving the learned variation out of the representation into a per-image 2D transform preserves the 3DGS render path.

## Preconditions & compatibility

Compatible with any rasterization-based radiance renderer with photometric drift (including SVRaster). The L1/D-SSIM split still lets appearance leak into geometry via SSIM's structure/contrast components — this is the concrete limitation [[ssim-decoupled-appearance-embedding_radl2026]] (CoMe) refines.

## Open questions

- Affine or γ-corrected transform variants showed only marginal gains — why does pixel-wise multiplication suffice?
- Strong view-dependent specular drift (wet surfaces, reflective glass) untested.
