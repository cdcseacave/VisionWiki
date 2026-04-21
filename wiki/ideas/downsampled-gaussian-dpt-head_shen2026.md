---
title: Downsampled Gaussian DPT head for compact feed-forward 3DGS
type: idea
source_paper: wiki/papers/shen2026_lyra2.md
also_in: []

scope: drop-in
stages: [generative-3d.shape-generation]
collapses: []
splits_into: []
rewrites: {}

inputs: [high-resolution-rgb-frame-or-video]
outputs: [downsampled-3dgs-attributes-per-k2-patch]
assumptions: [per-pixel-gaussian-predictor-head-architecture, high-resolution-input-makes-per-pixel-output-infeasible-at-streaming-budget]
requires_upstream_property: [dense-per-pixel-feature-map-from-pretrained-backbone]
requires_downstream_property: [rasterizer-accepts-sub-pixel-gaussian-density]
learned_params: [modified-dpt-head-with-k-stride-output]
failure_modes: [too-aggressive-downsampling-loses-thin-structures, k-parameter-must-be-retuned-per-resolution-regime, no-isolated-ablation-in-source-paper]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward, 3dgs, dpt-head, downsampling, depth-anything-v3, lyra-2, streaming, real-time-rendering]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Depth Anything v3 (DAv3), used by Lyra 2.0 as the feed-forward 3DGS backbone, originally predicts **one Gaussian per input pixel** through its Gaussian DPT head. On high-resolution generated videos this produces too many Gaussians for real-time rendering or streaming (each frame at 832×480 yields ~400K Gaussians; multiple frames exceed consumer-GPU rasterization budgets).

**The modification**: change the Gaussian DPT head to output a feature map downsampled by `k × k`. The network still **processes the original high-resolution image** through the encoder, preserving input signal fidelity; it just outputs one Gaussian per `k × k` patch. With `k = 2`, the predicted Gaussian count drops by `k² = 4×` while the per-Gaussian covariance naturally grows to cover the larger spatial footprint.

**Why this works** (design argument, no isolated ablation in the paper):
- The encoder's feature extraction still sees full-resolution evidence — fine details are encoded into the latent features even if the head doesn't emit a distinct Gaussian for each pixel.
- 3DGS rendering tolerates larger anisotropic Gaussians gracefully — the rasterizer is sub-pixel capable, and fewer-but-larger Gaussians often rival per-pixel-Gaussian output in perceived quality when the underlying features carry the signal.

Fine-tuning on Lyra-generated scenes (3,000 autoregressively-generated one-minute videos from DL3DV, 10K iterations, LR 5×10⁻⁵, batch 8) adapts DAv3 to the slightly-inconsistent multi-view character of generated data. This second part — fine-tuning on generated outputs — is **not novel to Lyra 2.0** (the precedent is Lyra 1 [2]); only the head modification is new here.

## Why it wins

**No isolated ablation** — this is a flagged weakness of the evidence. The paper reports `k=2` as the chosen configuration without ablating against `k=1` (per-pixel) or `k=3`. The argued benefits are:
- 4× fewer Gaussians per video frame → streaming-compatible representation.
- "More compact representation suitable for real-time rendering and data streaming" (§4.4).

**Downstream validation** (Table 2 Ours Full vs Ours + DAv3 on Tanks-and-Temples):
- LPIPS-G: 0.648 → 0.629 — the fine-tuned-for-generated-data pipeline clearly improves reconstruction quality on generated inputs.
- FID: 79.36 → 72.47.
- Subjective Quality: 14.42 → 18.80.

But these numbers *cannot be attributed to the head downsampling alone* — they combine the head modification, the fine-tuning on generated data (the Lyra 1 precedent), and the overall video-model upgrade. **The evidence for the head-downsampling idea in isolation is weaker than the other Lyra 2.0 ideas.**

## Preconditions & compatibility

- **Upstream**: any feed-forward architecture with a per-pixel Gaussian predictor head. DAv3 is the instance; the recipe transfers to similar heads (MVSplat-family, pixelSplat, NoPoSplat).
- **Downstream**: any 3DGS rasterizer. Modifies nothing about the rendering side.
- **Orthogonal to**: per-scene optimization (this idea lives strictly in the feed-forward regime), and to Gaussian-side compaction methods that prune after prediction (GGN, PixelGaussian — these post-hoc compress a predicted Gaussian set; the head-downsampling predicts a smaller set in the first place). The two approaches are compatible — predict fewer, then compact further.

## Trade-offs vs. the decomposed pipeline

Not applicable — `scope: drop-in`. Replaces a per-pixel DPT head with a strided variant.

## Open questions

- **Optimal `k` per resolution regime**: no ablation. `k=2` at 832×480 is chosen; unclear whether `k=3` or `k=4` would continue to preserve quality at lower Gaussian counts, or whether the quality cliff is near `k=2`.
- **Interaction with fine-tuning**: the paper couples head modification with fine-tuning on generated data. Whether the head modification alone (no fine-tune) already closes the streaming gap is untested.
- **Transferability to scene-scale vs object-scale**: Lyra tests at scene scale. On object-level feed-forward 3DGS (pixelSplat class), the aggressive down-sampling may lose thin structures (fingers, wires). Benchmark transfer untested.
- **Combine with post-hoc Gaussian compaction**: the `[[wang2026_feed-forward-3d-scene-modeling]] §4.3.2` family (GGN, PixelGaussian, FreeSplat++, LongSplat) all address the same streaming-budget concern via pruning. A combined predict-fewer-then-prune-further could compound. Candidate for a Pass B bet.
