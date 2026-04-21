---
title: Dual-smoothed fine-grained quantization for VGGT (Hadamard rotation + post-local channel smoothing)
type: idea
source_paper: wiki/papers/feng2025_quantvggt.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.numerical-precision]
collapses: []
splits_into: []
rewrites: {}

inputs: [trained-fp16-vggt-weights-and-activations]
outputs: [w4a4-quantized-weights-activation-scales]
assumptions: [vggt-family-architecture-with-special-tokens, post-training-transform-no-retraining, representative-calibration-data-available]
requires_upstream_property: [converged-training-checkpoint, identifiable-camera-and-register-special-tokens]
requires_downstream_property: [inference-runtime-supports-w4a4-mixed-precision-ops]
learned_params: [per-channel-scale-factors, hadamard-matrix-seed]
failure_modes: [non-vggt-architectures-without-special-tokens-see-smaller-gains, extreme-bit-widths-below-w4a4-require-per-token-calibration-not-addressed]

requires: []
unlocks: []
co_requires: [frame-aware-ptq-calibration-sampling_feng2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [vggt, quantization, ptq, training-free, efficiency, hadamard-rotation, w4a4]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
---

## Mechanism

A two-stage numerical transform applied to a trained VGGT checkpoint before W4A4 post-training quantization. Addresses VGGT's unique outlier structure: the first 5 tokens (camera token + 4 register tokens) are *data-independent* — their activations are nearly identical across inputs and produce heavy-tailed channel distributions that blow up any naïve quantization range.

**Stage 1 — Pre-Global Hadamard Rotation.** For each linear layer with weight `W ∈ ℝ^{d×d}` (where `d = 2ⁿ`), apply a random Hadamard transform `H` to both activations and weights before quantization:

$$XW^T = (XH)(WH)^T$$

By the central limit theorem, Hadamard rotation disperses outlier values across all `d` channels — turning a heavy-tailed distribution into an approximately Gaussian one (Lemma 3.1 in the paper). The transform is algebraically lossless (`H Hᵀ = I`), so the pre-quantization forward pass is unchanged. Only the post-quantization forward pass benefits: the rotated distribution is far more amenable to fixed-range INT4 quantization.

**Stage 2 — Post-Local Channel Smoothing.** Hadamard rotation removes inter-channel global skew but leaves within-channel variance. Compute per-channel scale factors derived *from the rotated distribution's max-abs values*:

$$c_i = \text{max}(|X^{\text{rot}}_i|)^\alpha / \text{max}(|W^{\text{rot}}_i|)^{1-\alpha}$$

where `X^rot = XH` and `W^rot = WH`. Key distinction from SmoothQuant (Xiao 2023): SmoothQuant derives `c_i` from the *pre-rotation* distributions; QuantVGGT derives from the *post-rotation* distributions. The rotated-distribution-derived scales are better-calibrated for the post-rotation quantization grid.

After both transforms, standard per-channel symmetric INT4 quantization is applied to weights and activations. The two-stage transform is required because neither alone handles VGGT's failure mode: Hadamard rotation alone (Fig. 3c) still leaves 2–3× within-channel variance; channel smoothing alone cannot touch the inter-channel outlier concentration from special tokens.

## Why it wins

Observation 1 (paper §3): VGGT's first 5 tokens have 10–100× larger activation magnitudes than regular patch tokens. Vanilla INT4 PTQ quantization gives these tokens most of the grid, wasting >90% of representational capacity on ~1% of tokens. Dual-Smooth remedies this directly.

Table 9 ablates the two stages independently:
- Vanilla SmoothQuant: collapses (AUC@5 drops from 60.6 FP16 → 14.2 W4A4).
- Pre-Global Rotation only: AUC@5 = 38.7.
- Pre-Global Rotation + Post-Local Smooth (full DSFQ): **AUC@5 = 55.7** (92% of FP16).

Comparison against QuaRot and SpinQuant (rotation-based PTQ baselines): DSFQ wins by 10–20 AUC points at W4A4 on all pose and pointmap benchmarks. The gain traces to the `co_requires` calibration data from [[frame-aware-ptq-calibration-sampling_feng2025]] — without stable calibration, the scale factors `c_i` become noisy.

Headline: W4A4 QuantVGGT retains ≥98% of FP16 accuracy on AUC@30, delivers **3.7× memory reduction** and **2.5× real-hardware speedup** on H100 inference. W8A8 variant retains 99.5%.

## Preconditions & compatibility

Consumes a trained VGGT checkpoint (FP16) + representative calibration data from the companion stage [[feed-forward-sfm.ptq-calibration-sampling]]. Produces W4A4 quantized weights and per-channel activation scales.

**Bundle constraint (`co_requires:`)**: [[frame-aware-ptq-calibration-sampling_feng2025]]. The headline W4A4 result requires both — the quantization transform and the calibration sampling must travel together. Partial adoption (DSFQ with uniform calibration, or NFDS with SmoothQuant) loses 5–15 AUC points in the paper's ablations.

Composition edges:
- **`co_requires: [frame-aware-ptq-calibration-sampling_feng2025]`** — bundle.
- **Composes freely (`unlocks:` implicit)** with [[vggt-token-merge-3-part-partition_shen2025]] and [[pooled-qk-block-sparse-global-attention_wang2025]] — quantization is orthogonal to both attention-mechanism and token-count transforms. All three can stack in theory (Bet #029).
- **`contradicts:`** nothing explicit. QuantVGGT's W4A4 does not interfere with dense / sparse / token-merged attention patterns because quantization operates at the linear-op level.

## Pipeline-shape implications

`scope: stage-swap`. The idea introduces *no* new topology — it fills the newly-defined stage [[feed-forward-sfm.numerical-precision]] with its dual-smoothed transform. Adopting pipelines gain a quantization node at deployment time; the trained forward pass DAG is unchanged.

## Trade-offs vs. the decomposed pipeline

Not applicable (no decomposition change). Standard PTQ trade-offs apply: accuracy floor of ~98% means tasks with tight accuracy requirements (e.g., millimeter-precision reconstruction) may prefer W8A8 (≥99.5% retention) over W4A4 (3.7× memory). Inference hardware support for W4A4 mixed-precision ops is required for the speedup to materialize — pure memory reduction is available on any platform.

## Open questions

- **Transferability to non-VGGT pointmap families**: CUT3R's recurrent state has different outlier structure; π³'s permutation-invariant design removes camera tokens. DSFQ's special-token-centered design may underperform on these.
- **Below W4A4 (W3A3, W2A2)**: paper does not test; the α-parameter tuning may need to be bit-width-aware.
- **Interaction with token merging**: FastVGGT's merged pseudo-tokens have different activation distributions than raw patch tokens. The Hadamard rotation is architecture-agnostic but the per-channel scales calibrated on unmerged tokens may not match. Joint DSFQ + FastVGGT calibration is Bet #029.
- **KV-cache quantization**: the paper quantizes static weights + per-pass activations but does not discuss quantizing the KV cache across blocks — relevant for very-long-sequence streaming settings.
