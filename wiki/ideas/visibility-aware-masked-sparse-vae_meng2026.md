---
title: Visibility-aware masked sparse VAE for partial TSDF scenes
type: idea
source_paper: wiki/papers/meng2026_seen2scene.md
also_in: []

scope: stage-swap
stages: [scene-completion.partial-scan-latent-encoding]
collapses: []
splits_into: []
rewrites: {}

inputs: [tsdf-volume-three-state-voxels]
outputs: [compact-latent-grid, per-latent-token-visibility-mask]
assumptions: [tsdf-representation, three-state-voxel-recoverable, bounded-scene-256-cubed-patch, voxel-size-1cm-class]
requires_upstream_property: [volumetric-fusion-emits-three-state-tsdf-with-sentinel-for-unobserved]
requires_downstream_property: [downstream-can-consume-or-ignore-visibility-mask]
learned_params: [sparse-conv-encoder-weights, sparse-conv-decoder-weights, learnable-empty-embedding-vector, kl-prior-scale]
failure_modes: [over-aggressive-masking-collapses-to-trivial-latent, learnable-embedding-overfits-to-bias-of-training-distribution, 8x-downsampling-loses-fine-surface-detail]

requires: []
unlocks: [partial-scan-encoding-without-empty-bias, visibility-aware-conditioning-for-other-models]
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sparse-vae, tsdf, visibility-aware, voxel, masked-encoding, fvdb, sparse-conv]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
validated_in: []
---

## Mechanism

A sparse-convolutional VAE (built on fVDB sparse residual blocks) that compresses a 256³ TSDF patch into an 8-channel latent grid with 8× spatial downsampling, with explicit handling of partial-observation structure.

**Three-state input voxel encoding** (recoverable from TSDF values):
- **surface**: |TSDF| < 3 × voxel_size — voxels near the zero-level set, observed.
- **empty**: TSDF ≥ truncation — free space, observed (camera saw past).
- **unknown**: TSDF = sentinel `−3 × voxel_size` — never observed by any camera.

**Encoder (`E_τ`)**:
1. *Structure-aware masking*: traverse the input volume; retain only voxels with state ∈ {surface, empty} (i.e., known voxels). Replace each unobserved voxel with a single *learnable empty embedding* vector before encoding. This ensures the encoder never sees "ambiguous" features for unobserved regions — it sees a fixed token saying "I never saw here".
2. Three stages of sparse residual blocks (GroupNorm → SparseConv → SiLU) with 2× max-pooling, growing features 64 → 128 → 256 → 512.
3. Bottleneck projection to mean + log-variance for KL-regularized latent of 8 channels at 32³ resolution (i.e., 8× downsampling of the 256³ input).
4. **Visibility mask propagation**: at each downsampling level, propagate the binary observed/unobserved status from input voxels to latent tokens. A latent token is "observed" if any source voxel in its receptive field was observed; otherwise unobserved.

**Decoder (`D_τ`)**:
1. Symmetric upsampling (nearest-neighbor) with sparse residual blocks.
2. At each resolution, a *structure head* prunes voxels predicted as empty (cuts compute on subsequent layers).
3. Two final prediction heads:
   - **Category head** (focal-loss BCE): classifies each surviving voxel as surface or empty.
   - **Geometry head** (tanh activation): predicts continuous TSDF value for surface voxels.

**Loss**: `L = L_KL + L_category + L_TSDF`, with the latter two computed only over **observed** voxels (visibility mask gates the loss).

The output for the downstream flow-matching stage is the latent grid `z` plus the per-latent-token visibility mask. The mask is the carrier that lets [[visibility-guided-masked-flow-matching_meng2026]] know which tokens to include in its loss.

## Why it wins

Table 3 of [Meng et al. 2026](../papers/meng2026_seen2scene.md) — the reconstruction column attributes:
- Without masked training, L1 ×10⁻⁴, L2 ×10⁻⁶, CD ×10⁻³ all degrade.

The causal story is the same as [[visibility-guided-masked-flow-matching_meng2026]] but applied at the autoencoder level: a vanilla sparse VAE trained on partial scans either (a) reconstructs unobserved voxels as "empty" (because that's what the data effectively says — empty = free space + unknown, indistinguishable in a 2-state representation), or (b) emits high-variance latent codes for unobserved regions because the encoder can't decide what to do with sentinel TSDF values. Structure-aware masking + the learnable empty embedding (a) factor unknown voxels out of the reconstruction loss entirely, and (b) give the encoder a fixed, low-variance representation to substitute for unknowns, so the latent prior is not contaminated.

The 8× downsampling + 8-channel latent compression is fairly aggressive — the resolution-loss flagged in §0.E of the paper traces here.

**Evidence note**: same caveat as [[visibility-guided-masked-flow-matching_meng2026]] — Table 3 ablates the *bundle* (encoder masking + flow-loss masking jointly), so the per-component evidence split is by metric type, not direct ablation.

## Preconditions & compatibility

**Upstream consumed**: a TSDF emitted from volumetric fusion (e.g., VDBFusion) that uses a sentinel value to encode unknown regions, or any equivalent 3-state voxel representation. KinectFusion-style 2-state TSDFs lose this information and would need to be re-fused with visibility tracking.

**Downstream imposed**: downstream consumers must either consume the visibility mask (as in [[visibility-guided-masked-flow-matching_meng2026]]) or be robust to the learnable-empty-embedding signal in unobserved latent regions. A naive downstream model that treats all latents uniformly will recover the empty-bias problem.

**Known compositions**:
- → bundle: [[visibility-guided-masked-flow-matching_meng2026]] (the only known downstream consumer of the visibility mask).
- → `unlocks:` visibility-aware conditioning for any voxel-feature pipeline (e.g., [[voxel-tsdf-multimodal-fusion_jatavallabhula2023]] currently treats unfused voxels and unobserved voxels identically — this idea suggests they should be distinguished).
- → cross-thread candidate for [[lifting-foundation-models-to-3d]]'s online-voxel-feature-fusion stage; see Bet on that thread.

## Pipeline-shape implications

Not required (`stage-swap`).

## Trade-offs vs. the decomposed pipeline

Not applicable.

## Open questions

- The learnable empty embedding is a single vector. Could it be made *spatially-conditioned* (e.g., positional-encoding-aware, so the model knows "I never saw here, but here is near a wall vs. here is in open space")? Connects to attention-based completion patterns.
- The 8× downsampling is a hard resolution ceiling. A multi-scale VAE (encoder pyramid → multiple latent grids) would give downstream flow matching access to multiple resolutions but at attention-cost.
- Visibility-mask propagation via "any source voxel observed" is a permissive rule (Boolean OR). A stricter rule ("majority observed" or "fraction-observed weighted") would give finer-grained gradient flow but might hurt large-region completion.
- Does this idea transfer to non-TSDF representations? E.g., a sparse point-cloud encoder, an occupancy-grid encoder, a Gaussian-splat encoder. The 3-state structure (occupied / known-empty / unknown) is generic, but the sentinel-TSDF carrier is TSDF-specific.
