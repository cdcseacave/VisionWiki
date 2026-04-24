---
title: Visibility-guided masked flow matching for 3D scene generation
type: idea
source_paper: wiki/papers/meng2026_seen2scene.md
also_in: []

scope: stage-swap
stages: [scene-completion.generative-prior]
collapses: []
splits_into: []
rewrites: {}

inputs: [partial-tsdf-latent, visibility-mask, layout-conditioning-tokens]
outputs: [generative-velocity-field-over-observed-tokens]
assumptions: [tsdf-representation, three-state-voxel-recoverable, bounded-scene, static-scene, indoor-domain]
requires_upstream_property: [visibility-mask-aligned-with-latent-grid, latent-encoding-preserves-three-state-voxel-info]
requires_downstream_property: [sampler-can-tolerate-undefined-velocity-on-unobserved-tokens]
learned_params: [sparse-dit-velocity-network-weights, layout-token-embedding]
failure_modes: [mask-leakage-causes-empty-bias-in-unobserved-regions, classifier-free-guidance-overweights-layout-on-cluttered-scenes, indoor-trained-priors-fail-out-of-distribution-on-outdoor]

requires: []
unlocks: [partial-data-trained-3d-generators-as-priors-for-other-tasks]
co_requires: [visibility-aware-masked-sparse-vae_meng2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [flow-matching, masked-loss, 3d-generation, scene-completion, partial-data, real-data-training]
created: 2026-04-24
updated: 2026-04-24
status: unclaimed
validated_in: []
---

## Mechanism

Standard conditional flow matching learns a velocity field `v_θ(z_t, t, c)` to transport noise `z_0 ~ p_0` to data `z_1 ~ p_data` via the loss

$$\mathcal{L}_{\mathrm{gen}} = \mathbb{E}_{z_0 \sim p_0,\, z_1 \sim p_{\mathrm{geo}},\, t \sim \mathcal{U}[0,1]}\left[\| v_\theta(z_t, t, c) - (z_1 - z_0) \|^2\right],$$

with `z_t = (1−t) z_0 + t z_1`. The novel modification is **token-level masking of the loss expectation**: for each geometry latent token, look up the corresponding source TSDF voxel; if that voxel was *unobserved* (sentinel value `−3 × voxel_size` from volumetric fusion), exclude that token from the loss. The model receives no gradient on unobserved tokens, so it never learns the wrong "unobserved → empty" mapping that would arise if all tokens were treated uniformly.

The visibility mask is *not* an architectural mask on attention — self-attention is full / dense over all geometry + layout tokens, so unobserved tokens still attend and are attended to. The mask only gates *which tokens contribute to the loss*. At inference, the model emits velocity predictions for all tokens (including unobserved ones), and these are used by the sampler — the model's behavior on unobserved tokens at sampling time is shaped purely by what attention learned in observed regions, not by direct supervision.

Classifier-free guidance: the layout condition `𝓑` is randomly dropped during training, making it optional at inference and enabling guidance-scale-controlled sampling.

## Why it wins

Table 3 of [Meng et al. 2026](../papers/meng2026_seen2scene.md) ablates *masked training* (which jointly toggles this idea + [[visibility-aware-masked-sparse-vae_meng2026]]). Without masked training:
- Reconstruction L1 ×10⁻⁴, L2 ×10⁻⁶, CD ×10⁻³ all degrade sharply (autoencoder side, attributable to the VAE half of the bundle).
- Generation U3D-FPD degrades and VLM perceptual score drops (flow-matching side, attributable to this idea).

Causal story: a generative model trained on partial scans without masking learns the empirical conditional `p(geometry | observed_voxels) ≈ p(geometry | observed=v, unobserved=empty)` — i.e., it learns to fill cavities with *whatever the dataset's depth-shadow geometry actually looks like* (always empty, because the camera saw past the surface). This is a *biased* estimate of the true scene distribution. With masking, the loss is computed only over genuinely observed regions, so the velocity field models the *true* `p(observed_geometry | layout)`, and at inference the model can extend that distribution into unobserved regions by way of the layout signal + attention to neighboring observed tokens.

**Evidence note**: the ablation conflates this idea with [[visibility-aware-masked-sparse-vae_meng2026]] — the paper does not separately ablate VAE-only masking vs flow-only masking. The split-attribution above is reasoned from the metric type (reconstruction = VAE; generation = flow) but is not directly demonstrated. Bets predicated on this idea in isolation should acknowledge the joint-evidence caveat.

## Preconditions & compatibility

**Upstream consumed**: a latent encoding that (a) preserves enough information about which source voxels were observed to recover a per-latent-token visibility mask, and (b) does not bake "unobserved → empty" into the latent prior. [[visibility-aware-masked-sparse-vae_meng2026]] provides both — hence the `co_requires:`.

**Downstream imposed**: the sampler must tolerate "junk" velocity predictions on unobserved tokens during the early sampling steps. In Seen2Scene this works because the layout signal + attention to observed neighbors guides the sampler toward plausible geometry; without strong layout conditioning, the unobserved regions could drift. Adopting this idea in a different downstream context (e.g., a non-layout-conditional generator) requires an alternative guidance signal for unobserved regions.

**Known compositions**:
- → `unlocks:` partial-data-trained 3D generative priors generally — any flow-matching or score-matching loss on data with structured missingness can adopt token-level loss masking.
- → bundle: [[visibility-aware-masked-sparse-vae_meng2026]] (`co_requires:`).

## Pipeline-shape implications

Not required (scope is `stage-swap`, not topology-changing). The idea slots into [[scene-completion.generative-prior]] without modifying surrounding nodes.

## Trade-offs vs. the decomposed pipeline

Not applicable (`stage-swap` scope).

## Open questions

- Does the trick generalize to score-matching / DDPM losses, or is it specific to flow-matching's straight-line interpolant `z_t = (1−t) z_0 + t z_1`? The interpolant linearity is what lets "skip this token" work cleanly; non-linear interpolants might leak gradient via the attention path.
- Optimal CFG scale is 3.0 for layout-conditional generation. For pure scan-completion (Stage 4 ControlNet path), should CFG be higher (more layout trust) or lower (more partial-scan trust)? Not addressed.
- The visibility mask is *binary* (observed / unobserved). Could a *weighted* mask (e.g., observation-confidence from depth-sensor reliability or angle-of-incidence) further improve the generative prior? Connects to [[per-pixel-probabilistic-view-selection_schonberger2016]] and [[depth-proportional-uncertainty-fusion_pataki2025]] — uncertainty-weighted masked flow matching is an unexplored direction.
- House-scale completion uses independent 256³ patches with weighted-average overlap fusion. Does the masked-flow-matching loss extend to *globally-attended* generation across patches, or does the attention complexity make this infeasible? Open.
