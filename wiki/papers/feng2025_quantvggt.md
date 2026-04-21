---
title: "Quantized Visual Geometry Grounded Transformer (QuantVGGT)"
type: paper
tags: [vggt, quantization, ptq, training-free, efficiency, hadamard-rotation, w4a4, iclr-2026]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/fundamentals/feng_2025_quantvggt.pdf
url: https://arxiv.org/abs/2509.21302
code: https://github.com/wlfeng0509/QuantVGGT
license_paper: CC-BY-4.0
license_code: Apache-2.0
status: draft
---

📄 [Full paper](../../papers/fundamentals/feng_2025_quantvggt.pdf) · [arXiv](https://arxiv.org/abs/2509.21302) · [code](https://github.com/wlfeng0509/QuantVGGT) · ICLR 2026

_Paper license: `CC-BY-4.0` · Code license: `Apache-2.0` — commercial-use clean on both_

## TL;DR

First post-training quantization (PTQ) framework for VGGT-class feed-forward 3D reconstruction models. W4A4 (4-bit weights + 4-bit activations) delivers **3.7× memory reduction and 2.5× real-hardware speedup on H100** while retaining ≥98% of FP16 accuracy across pose and pointmap benchmarks. Two technical contributions attack VGGT-specific failure modes: **Dual-Smoothed Fine-Grained Quantization (DSFQ)** handles VGGT's heavy-tailed activation distributions caused by data-independent special tokens (camera + register), via pre-global Hadamard rotation + post-local channel smoothing; **Noise-Filtered Diverse Sampling (NFDS)** stabilizes calibration-sample selection with deep-layer statistics filtering + frame-aware clustering.

## Problem

Generic PTQ methods (SmoothQuant, QuaRot, SpinQuant) break on VGGT at W4A4: AUC@5 collapses to 14.2 from 60.6 FP16. Two VGGT-specific failure modes:

1. **Heavy-tailed special-token activations.** VGGT's first 5 tokens (1 camera + 4 register) are *data-independent* — their activations are nearly identical across inputs and 10–100× larger in magnitude than patch tokens. Naïve quantization grids waste >90% of representational capacity on these ~1% of tokens, leaving regular patches severely under-quantized.

2. **Unstable calibration-sample selection.** The multi-view nature of VGGT inputs means that uniform random calibration sampling yields high per-run variance — best-seed vs first-seed W4A4 results can differ by 10+ AUC points. This is a reproducibility problem on top of the accuracy problem.

Neither failure mode appears in LLM PTQ literature; both are specific to multi-view transformers with persistent special tokens.

## Method

### DSFQ — Dual-Smoothed Fine-Grained Quantization (see [[vggt-dual-smoothed-quantization_feng2025]])

Two-stage numerical transform applied before standard per-channel INT4 quantization:

**Stage 1 — Pre-Global Hadamard Rotation.** For each linear layer `y = XWᵀ`, apply an algebraically-lossless Hadamard transform:

$$X W^T = (XH)(WH)^T$$

where `H` is a random Hadamard matrix (`H Hᵀ = I`). By the central limit effect, the rotated distribution `XH` becomes approximately Gaussian, spreading the special-token outliers across all `d` channels (Lemma 3.1). The heavy-tailed distribution is flattened without changing the forward-pass value.

**Stage 2 — Post-Local Channel Smoothing.** Hadamard rotation spreads inter-channel outliers but leaves within-channel variance. A channel-wise scale normalizes the remaining variance:

$$c_i = \frac{\max(|X^{\text{rot}}_i|)^\alpha}{\max(|W^{\text{rot}}_i|)^{1-\alpha}}$$

Key distinction from SmoothQuant: the `max` values are taken from the *post-rotation* distributions, not pre-rotation. This matches the scale factors to the quantization grid that will actually operate on the rotated data.

### NFDS — Noise-Filtered Diverse Sampling (see [[frame-aware-ptq-calibration-sampling_feng2025]])

Two-part calibration-set curation:

1. **Noise filtering**: candidate sequences whose deep-layer (penultimate aggregator block) max-abs activations fall in the top/bottom ~5% are discarded as "noisy extremes" — textureless walls, degenerate viewpoints, etc. These skew the quantization range estimate.

2. **Frame-aware clustering**: remaining samples clustered using *per-frame* activation-distribution features (mean + variance + quantile statistics across the frames of a multi-view sample). K-means on these frame-distribution features yields clusters grouped by multi-view distributional similarity rather than per-image appearance. Final calibration pool drawn uniformly across clusters.

DSFQ and NFDS ablate **co-dependently**: DSFQ alone (with random calibration) = 38.7 AUC@5; NFDS alone (with SmoothQuant) = 22.1 AUC@5; full DSFQ+NFDS = 55.7 AUC@5 (~92% of FP16). Hence the `co_requires:` bundle on both idea pages.

## Results

| Config | AUC@30 | AUC@15 | AUC@5 | Memory | H100 Latency |
|---|---|---|---|---|---|
| VGGT FP16 | 85.5 | 73.2 | 60.6 | 1.0× | 1.0× |
| SmoothQuant W4A4 | 68.1 | 40.3 | 14.2 | 3.7× | 2.5× |
| QuaRot W4A4 | 72.4 | 50.1 | 29.5 | 3.7× | 2.5× |
| SpinQuant W4A4 | 75.8 | 55.2 | 35.7 | 3.7× | 2.5× |
| **QuantVGGT W4A4** | **84.1** | **68.9** | **55.7** | **3.7×** | **2.5×** |
| QuantVGGT W8A8 | 85.3 | 72.8 | 60.2 | 1.9× | 1.4× |

Consistent ~98% accuracy retention across RealEstate10K, CO3D, TUM, ScanNet, 7-Scenes, NRGBD, DTU, ETH3D. The **2.3–3.1 AUC point stability gain** from NFDS (Table 10) is under-reported relative to the DSFQ gains but is the contribution most likely to transfer to non-VGGT PTQ pipelines (monocular-depth foundation models, dense matchers).

## Why it matters

Populates the **numerical precision** axis of the VGGT-compression family (alongside [[shen2025_fastvggt|FastVGGT]] on token count and [[wang2025_faster-vggt-block-sparse|Faster-VGGT block-sparse]] on attention sparsity). Quantization is orthogonal to both attention-mechanism and token-count transforms — QuantVGGT's 3.7× memory × 2.5× speed composes with the other two axes in theory.

The **commercial-license profile** (CC-BY-4.0 paper, Apache-2.0 code) makes QuantVGGT the most deployment-ready of the three. FastVGGT inherits Meta's VGGT License (research-only); Faster-VGGT has no license declared. QuantVGGT is clean for commercial adoption — a quiet but important ingest finding given the thread's bet-level commercial-readiness tracking in [wiki/meta/license-audit.md](../meta/license-audit.md).

## Pipeline contribution

Two co-required ideas (bundle constraint) and two new stages:

- [[vggt-dual-smoothed-quantization_feng2025]] · candidate thread: [[feed-forward-structure-from-motion]] · op_target: op:default (Tier 3) · mechanism: pre-global Hadamard rotation + post-local channel smoothing to handle special-token heavy tails.
- [[frame-aware-ptq-calibration-sampling_feng2025]] · candidate thread: [[feed-forward-structure-from-motion]] · op_target: op:default (Tier 3) · mechanism: deep-layer noise filtering + frame-aware clustering for PTQ calibration samples.
- Introduces stages [[feed-forward-sfm.numerical-precision]] and [[feed-forward-sfm.ptq-calibration-sampling]] as deployment-time transform slots.

## Relation to prior work

- Builds on [[vggt|VGGT]] backbone.
- Extends rotation-based PTQ (QuaRot, SpinQuant — Ashkboos 2024, Zhao 2024) with the post-rotation-derived channel smoothing specific to VGGT's special-token structure.
- Distinct from SmoothQuant (Xiao 2023) — SmoothQuant's α-ratio formula is derived from pre-rotation distributions; QuantVGGT derives from post-rotation distributions.
- Orthogonal to [[shen2025_fastvggt|FastVGGT]] (token count) and [[wang2025_faster-vggt-block-sparse|Faster-VGGT block-sparse]] (attention sparsity) — together, the three form the VGGT-compression family named by [[wang2026_feed-forward-3d-scene-modeling]] §4.3.

## Open questions / limitations

- **Below W4A4**: no W3A3 or W2A2 results reported. The α-parameter in channel smoothing may need to be bit-width-aware.
- **Transfer to non-VGGT**: CUT3R, Fast3R, π³ — the special-token assumption varies. π³ removes camera tokens entirely; DSFQ's heavy-tail remedy becomes less motivated.
- **KV-cache quantization across blocks**: paper quantizes static weights + per-pass activations but does not discuss KV-cache compression for streaming long-sequence settings.
- **Interaction with FastVGGT's token merging**: merged pseudo-tokens have different activation statistics than raw patch tokens; the calibration set composed from unmerged inputs may not calibrate the merged-input regime correctly. Untested — part of Bet #029.
- **NFDS's transfer to non-multi-view pipelines**: the frame-aware clustering premise assumes multi-view-structured inputs. For monocular pipelines (single-image depth, monocular pointmap), the clustering degenerates to per-image — likely still useful for the noise filter (Part 1) alone.

## Code & license

- **Paper license**: `CC-BY-4.0` — commercial-use clean.
- **Code license**: `Apache-2.0` — commercial-use clean.
- **Caveat**: the base VGGT checkpoint QuantVGGT quantizes still carries Meta's VGGT License (non-commercial research-only). Commercial deployment requires either retraining a VGGT-equivalent on commercial-safe data or negotiating VGGT-license terms with Meta. QuantVGGT's contribution is commercial-clean; the underlying model is not.

## References added to the wiki

- [[vggt-dual-smoothed-quantization_feng2025]] (new idea)
- [[frame-aware-ptq-calibration-sampling_feng2025]] (new idea, `co_requires` the above)
- [[feed-forward-sfm.numerical-precision]] (new stage)
- [[feed-forward-sfm.ptq-calibration-sampling]] (new stage)
- Updated [[feed-forward-structure-from-motion]] (thread: Tier 3 evidence, capability gap closed, Bets #029/#030)
