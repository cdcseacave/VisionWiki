---
title: Scene-completion generative prior
type: stage
slug: scene-completion.generative-prior
consumes: [latent-grid, visibility-mask, optional-layout-condition]
produces: [generative-velocity-field-or-score-over-latent-tokens]
invariants: [unobserved-tokens-not-supervised-during-training, generated-distribution-matches-observed-region-distribution]
provides_properties: [stochastic-multi-sample-output, classifier-free-guidance-controllable]
requires_upstream_properties: [observed-vs-unobserved-distinguishable-at-latent-resolution, latent-grid-spatially-aligned-with-source-volume]
data_regime: [partial-data-trainable, bounded-scene]
tags: [scene-completion, flow-matching, generative-prior, masked-loss, sparse-dit]
created: 2026-04-24
updated: 2026-04-24
---

## What it is

The core generative model that learns the distribution of 3D scene geometry from partial observations. Outputs a velocity field (flow-matching) or score (diffusion) over latent geometry tokens, conditioned on an optional layout signal.

The defining requirement: the loss must mask unobserved tokens. A vanilla generative prior trained on partial-data latents without masking would learn the *biased* conditional `p(geometry | observed=v, unobserved=empty)` rather than the true `p(observed_geometry | layout)`, and at inference would over-confidently emit "empty" for unobserved regions.

This stage covers the *pretrained* generative prior (no completion-specific fine-tuning yet — that's the next stage). The output of this stage at sampling time is unconditional or layout-conditioned scene generation from scratch.

## Example fillers

- [[visibility-guided-masked-flow-matching_meng2026]] — sparse DiT (28 blocks, RoPE) with per-token masked flow-matching loss. Forms a `co_requires:` bundle with [[visibility-aware-masked-sparse-vae_meng2026]] (the upstream encoder that provides the visibility mask).
- [[clip-painted-3d-layout-conditioning_meng2026]] — the conditioning signal injection mechanism (paints CLIP-encoded box labels into a 3D map aligned with the latent grid; tokenized into the joint self-attention).

These two ideas naturally co-occur in this stage in Seen2Scene's pipeline; they're separable in principle (the masked flow matching could use a different layout encoding, and the CLIP-painted layout could feed an unmasked baseline) but the current SOTA filler combines both.

## Notes on valid fillers

Valid fillers must:
1. Apply some form of masked or otherwise visibility-aware loss during training.
2. Be amenable to conditional sampling (for downstream completion fine-tuning to work).
3. Produce a *stochastic* output (multi-sample diversity is what enables generative completion to outperform regression-style baselines like SG-NN/NKSR).

A regression-style scene generator (deterministic, no stochastic sampling, no classifier-free guidance) would not fill this stage — the downstream condition-injection stage's ControlNet recipe assumes a generative prior.
