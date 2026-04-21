---
title: Canonical-coordinate warping as appearance-neutral correspondence injection
type: idea
source_paper: wiki/papers/shen2026_lyra2.md
also_in: []

scope: stage-swap
stages: [video-world.correspondence-injection]
collapses: []
splits_into: []
rewrites: {}

inputs: [retrieved-frames, per-frame-depth, camera-intrinsics, camera-extrinsics, target-camera]
outputs: [per-token-correspondence-embedding]
assumptions: [static-scene, known-intrinsics, pretrained-dit-value-projection-worth-preserving, full-resolution-depth-reliable]
requires_upstream_property: [retrieved-frames-with-per-frame-depth, pretrained-transformer-backbone]
requires_downstream_property: [self-attention-blocks-accept-additive-bias-on-q-and-k]
learned_params: [aggregation-mlp-zero-init, sinusoidal-positional-encoding-parameters]
failure_modes: [severe-disocclusion-produces-empty-canonical-map-slots, depth-noise-produces-wrong-correspondences, Q-K-injection-without-V-may-underfit-on-novel-geometric-configurations-not-seen-in-pretraining]

requires: [per-frame-3d-cache-retrieval_shen2026]
unlocks: []
co_requires: [per-frame-3d-cache-retrieval_shen2026]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [video-generation, correspondence-injection, canonical-coordinates, forward-warp, dit-conditioning, attention-injection, lyra-2, information-routing]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Given the `N_s = 5` retrieved history frames from `[[per-frame-3d-cache-retrieval_shen2026]]` — each with its own depth `D_j^s` and camera `(T_j^s, K_j^s)` — convey their geometric relation to the target viewpoint by:

1. **Construct a canonical coordinate map** for each retrieved frame `j`:
$$C_j \in [-1, 1]^{3 \times H \times W}, \quad C_j(h, w) = \left(u, v, \ 2 \cdot \tfrac{j}{N_s} - 1\right)$$
where `(u, v)` are normalized spatial coordinates and the third channel encodes a frame-index scalar per retrieved slot.

2. **Forward-warp** `C_j` into the target viewpoint using the retrieved frame's full-resolution depth:
$$\hat{C}_j = \text{FwdWarp}(C_j, \ D_j^s, \ T_j^s, \ T^*, \ K_j^s, \ K^*)$$
and stack the warped depth as a fourth channel, yielding `[\hat{C}_j; \hat{D}_j] ∈ ℝ^{4 \times H \times W}`.

3. **Encode** the 4-channel correspondence map: pixel-shuffle to match DiT latent spatial resolution → sinusoidal positional encoding per channel → pixel-shuffle + single linear → per-token embedding `p`.

4. **Inject on queries and keys only** (not values) at every DiT self-attention block:
$$q = W_Q(x + p), \quad k = W_K(x + p), \quad v = W_V(x)$$
Padded slots (when fewer than `N_s` frames are retrieved) are zero-canonical-valued so the model can distinguish real correspondences from empty slots.

**Why warp canonical coords and not RGB.** Warped RGB from a history frame inevitably contains disocclusion holes, stretching artifacts, and depth-boundary bleeding. The paper's explicit argument (§4.2 "In summary"): if the DiT conditions on warped RGB, "the video model tends to re-generate these artifacts — the warped image acts as a crutch that bypasses the generative prior rather than informing it." Canonical coordinates carry the *same geometric correspondence information* (where does target pixel `(h, w)` come from in frame `j`?) without any appearance content, so appearance synthesis stays entirely in the DiT's learned pixel prior.

**Why Q/K only, not V.** Injecting on the value projection would directly perturb the pretrained value distribution, destabilizing the learned pixel prior. Q/K injection changes *which tokens attend to which* without changing *what information each token carries* at the attention output. The aggregation MLP is zero-initialized so training starts at pretrained Wan 2.1 behavior and additively learns the correspondence guidance.

## Why it wins

**Partial isolation** (Table 3 "w/ Explicit Corr. Fusion"): replacing the learned MLP aggregation with hard depth-reasoning-based fusion drops Camera Controllability **63.87 → 57.29 (−6.58)** — showing that learned aggregation handles noisy depth gracefully where hard geometric fusion breaks. This ablates the aggregation mechanism but **does not cleanly isolate canonical-coord-vs-warped-RGB** — that's the gap in the ablation evidence.

**Bundle isolation with Idea 1** (Table 3 "w/ Global Point Cloud"): replacing both the per-frame retrieval and the correspondence injection with a single global-point-cloud rendered-conditioning drops Camera Controllability **−14.01**. The bundle wins decisively; attributing shares between retrieval and injection requires a finer ablation the paper does not run.

**Design-level evidence** (qualitative §4.2 reasoning): the paper motivates canonical-coord-over-RGB on first principles (artifact propagation), and this aligns with a broader architectural pattern for diffusion conditioning — injecting geometric structure without pixel content lets the prior do what it was trained for.

## Preconditions & compatibility

- **Upstream**: must consume retrieved history frames with per-frame depth and camera parameters. This is why `requires` and `co_requires` both include `[[per-frame-3d-cache-retrieval_shen2026]]`: the correspondence map requires per-frame geometry, which only the per-frame cache provides (a global fused cloud loses per-frame provenance).
- **Backbone**: assumes a pretrained Transformer-family backbone where Q/K-side additive injection is meaningful (standard for ViT, DiT, Wan 2.1). Transferable in principle to any diffusion backbone with self-attention blocks.
- **Zero-init caveat**: the aggregation MLP is zero-initialized so training starts from pretrained behavior. This choice assumes the pretrained backbone is already "almost right" for short-horizon video generation — only the correspondence guidance is additively learned. Training from scratch with this injection is not tested.

## Trade-offs vs. the decomposed pipeline

Not applicable — `stage-swap` scope. The stage `[[video-world.correspondence-injection]]` admits multiple filler families (warped-RGB, pose-embedding-only, canonical-coord). Lyra 2.0 argues for and instantiates the canonical-coord filler; no pipeline decomposition is sacrificed.

## Open questions

- **Isolation of canonical-coord-vs-warped-RGB**: as flagged above, not cleanly ablated. The paper's argument is qualitative + first-principles; a direct A/B would strengthen the idea's empirical claim.
- **Transferability beyond Wan 2.1 / video**: the Q/K-only zero-init injection recipe is a general pattern for "bolting structured guidance onto a pretrained foundation model without destabilizing it." It is a candidate for cross-thread adoption (e.g. injecting depth/normal priors into SD3 or other pretrained diffusion backbones). Untested here.
- **Handling severe disocclusion**: if many retrieved pixels have no correspondence in the target view, the canonical map is mostly padded. The paper reports padded slots signal "no real correspondence" but does not measure how graceful degradation is when coverage drops below, say, 30%.
- **Bridge bundle check**: `[[per-frame-3d-cache-retrieval_shen2026]]` co-requires this idea. Partial adoption (retrieval without canonical-coord injection) would leave the DiT to infer correspondences from pixels — an unmeasured regime in this paper; Lyra 2.0 does not evaluate retrieval-only without correspondence warping.
