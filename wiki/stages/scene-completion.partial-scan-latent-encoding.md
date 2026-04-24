---
title: Scene-completion partial-scan latent encoding
type: stage
slug: scene-completion.partial-scan-latent-encoding
consumes: [partial-tsdf-volume-three-state-voxels]
produces: [compact-latent-grid, per-latent-token-visibility-mask]
invariants: [visibility-information-preserved-from-input-to-latent, latent-prior-not-biased-toward-empty-on-unobserved-regions]
provides_properties: [observed-vs-unobserved-distinguishable-at-latent-resolution, latent-grid-spatially-aligned-with-source-volume]
requires_upstream_properties: [tsdf-uses-sentinel-value-for-unobserved-voxels-or-equivalent-3-state-encoding]
data_regime: [bounded-scene-256-cubed-patch, voxel-size-1cm-class, indoor-or-outdoor]
tags: [scene-completion, sparse-vae, tsdf, visibility-aware, latent-encoding]
created: 2026-04-24
updated: 2026-04-24
---

## What it is

Encodes a partial 3D scan (TSDF or equivalent 3-state voxel grid) into a compact latent grid suitable for downstream generative modeling, *while preserving the distinction between observed and unobserved regions*.

The defining requirement: a vanilla autoencoder collapses the 3-state structure (surface / empty / unknown) into a 2-state latent (effectively, "the latent says this voxel will be reconstructed as some TSDF value"). This stage explicitly preserves the third state in two ways: (a) the encoding never confuses unknown for empty during training, and (b) a per-latent-token visibility mask is emitted alongside the latent grid so downstream stages can apply masked losses.

## Example fillers

- [[visibility-aware-masked-sparse-vae_meng2026]] — the sparse-conv VAE with structure-aware masking + learnable empty embedding; emits a 32³ × 8-channel latent grid + per-token visibility mask from a 256³ input. Forms a `co_requires:` bundle with [[visibility-guided-masked-flow-matching_meng2026]].

## Notes on valid fillers

A filler is valid here iff it:
1. Accepts a 3-state voxel input (or recovers the 3-state structure from raw TSDF + sentinel encoding).
2. Emits a latent representation that preserves enough information for downstream visibility-mask reconstruction.
3. Does not bake "unobserved → empty" or "unobserved → arbitrary noise" into the latent prior — both are subtly destructive.

A vanilla sparse VAE (no visibility awareness) is *not* a valid filler — it produces a biased latent prior that downstream generators will inherit. Same for any autoencoder that drops the visibility mask before the bottleneck.

Alternative architectures (point-cloud encoders, Gaussian-splat encoders) could fill this stage if they respect the same invariants, but no current wiki idea does so.
