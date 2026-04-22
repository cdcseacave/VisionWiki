---
title: "C-RADIOv4 (Tech Report)"
type: paper
tags: [vision-foundation-model, knowledge-distillation, multi-teacher, agglomerative-vfm, vision-transformer, shift-equivariance, vitdet, commercial-license]
created: 2026-04-22
updated: 2026-04-22
sources: [wiki/papers/heinrich2025_radiov25.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/carion2026_sam-3.md]
local_paper: papers/fundamentals/ranzinger_2026_c-radiov4.pdf
url: https://arxiv.org/abs/2601.17237
code: https://github.com/NVlabs/RADIO
license_paper: arxiv-nonexclusive
license_code: nvidia-open-model-license
status: draft
---

📄 [Full paper](../../papers/fundamentals/ranzinger_2026_c-radiov4.pdf) · [arXiv](https://arxiv.org/abs/2601.17237) · [code](https://github.com/NVlabs/RADIO) · weights: [C-RADIOv4-H](https://huggingface.co/nvidia/C-RADIOv4-H) · [C-RADIOv4-SO400M](https://huggingface.co/nvidia/C-RADIOv4-SO400M)

_Paper license: `arxiv-nonexclusive` · Code/weights license: `nvidia-open-model-license` (commercial use permitted)_

## TL;DR

C-RADIOv4 is the latest iteration of NVIDIA's agglomerative vision foundation model family, refining [[heinrich2025_radiov25|RADIOv2.5]] with a 2026 teacher set (SigLIP2-g-384, DINOv3-7B, SAM3) and four training-side upgrades: stochastic resolution training (10 sampled resolutions instead of 2), a shift-equivariant spatial-feature loss that breaks teachers' fixed-pattern positional noise, shift-equivariant MESA for EMA self-matching, and an angle-normalized summary loss that extends PHI-S's dispersion balancing from spatial to summary tokens. Adds **ViTDet mode** for drastic high-resolution inference speedup, and ships SO400M (412M) and H (631M) variants under the NVIDIA Open Model License, making it the first commercially-usable RADIO. Matches DINOv3-7B on dense tasks at ~10× fewer parameters.

## Problem

[[heinrich2025_radiov25|RADIOv2.5]] closed the "mode-switch" between low- and high-resolution features but left four issues on the table: (1) its NSCL license blocked commercial integration; (2) two-resolution training produced a piecewise-smooth scaling curve with degradation at unusual resolutions; (3) student distillation mimicked *positional noise* from the teachers (DINOv2/3 register tokens, SAM ViTDet window borders) rather than only their semantics; (4) cosine-distance summary-loss biased the student toward teachers with larger angular dispersion, starving smaller-dispersion teachers like SigLIP2. At deployment, high-resolution inference was quadratic in tokens — painful for VLMs at 1024px+.

## Method

### 2026 teacher set

DFN-CLIP + DINOv2 + SAM is replaced with **SigLIP2-g-384 + DINOv3-7B + SAM3**. Rationale: SigLIP2 is the frontier text-image encoder, being adopted widely (Qwen3-VL); DINOv3 pushes SSL dense representations; SAM3 replaces SAM with "concepts" capability. DFN-CLIP is dropped.

### Stochastic resolutions (§2.2)

Instead of two discrete training resolutions (RADIOv2.5), uniformly sample each batch from **{128, 192, 224, 256, 384, 432}** (low-res partition) and **{512, 768, 1024, 1152}** (high-res partition). Low-res uses raw teacher outputs. High-res uses **FeatSharp 3× upsampling** on SigLIP2's fixed 384px output to align it with the student's up-to-1152px grid. SAM3 uses mosaic augmentation (it only supports 1152×1152 input). Result: smoother zero-shot and kNN accuracy curves across 200–1100px.

### Shift-equivariant loss (§2.3.1) — the headline mechanism

The student and each teacher independently see a random crop-shift of the image (shifts in patch-size increments, so no interpolation). Track a mapping $\mathcal{F}_{S \to T}$ from student coordinates to teacher coordinates. Loss evaluated only on the set $\Omega$ of shared spatial positions:

$$L_{\text{spatial}}(x, \hat{y}) = \frac{1}{|\Omega|} \sum_{u \in \Omega} \big(\mathcal{F}_{S \to T}[x]_u - \hat{y}_u\big)^2$$

where $x$ is student output, $\hat{y}$ is PHI-S-normalized teacher output. Because teachers' positional artifacts (register tokens, ViTDet window borders) live in *teacher-relative* coordinates but the student sees *student-relative* coordinates, the only consistent signal under random shift is semantics. Positional noise can't be learned.

### Shift-equivariant MESA (§2.3.2)

MESA (Du et al. 2022) matches the EMA of student weights to the student output — a sharpness-aware trick. C-RADIOv4 applies MESA across *different crops* of student vs EMA-student, aligned via $\mathcal{F}_{S \to S}$:

$$L_{\text{mesa}}(x, \tilde{x}) = \frac{1}{|\Omega|} \sum_{u \in \Omega} \big(\mathcal{F}_{S \to S}[\text{LN}(x)]_u - \text{LN}(\tilde{x})_u\big)^2$$

LN is layer-norm without learnable affine.

### Angle-normalized (balanced) summary loss (§2.5)

PHI-S normalized the magnitudes of spatial features but didn't touch summary tokens (they were cosine-matched). Observation: per-teacher summary features fall into a *cone* of different radii. SigLIP2 angular dispersion 0.694; DINOv3-H+ 2.120; DINOv3-7B 2.186 — ~3× apart (Table 3). Large-dispersion teachers produce larger losses and dominate the gradient. Fix: replace cosine distance with angular distance normalized by the teacher's own angular dispersion:

$$\mu_y = \frac{\mathbb{E}[y]}{\|\mathbb{E}[y]\|}, \quad \text{Disp}(\Theta_y) = \mathbb{E}[\Theta(y, \mu_y)^2], \quad L_{\text{angle}}(x,y) = \frac{\Theta(x,y)^2}{\text{Disp}(\Theta_y)}$$

This is PHI-S generalized from spatial-feature magnitudes to summary-feature angles.

### DAMP (§2.4)

Multiplicative weight noise during training (Du et al. 2020) for sharpness-aware robustness. Not novel, used as tool.

### ViTDet mode (§3.2) — deployment-side

Plain Vision Transformer for object detection (Li et al. 2022): most transformer blocks use windowed attention (6×6 to 32×32 token windows), with 4 global-attention layers interspersed. C-RADIOv4 supports a `vit_det_window_size` flag at construction time. Window size × patch size must divide input resolution. Figure 5/9: dramatic latency reduction at high resolution (SO400M-VDT12 is faster than SAM3's ViT-L+; H-VDT8 nearly as fast). Quality loss at small windows is slight.

### Variants

- **C-RADIOv4-SO400M**: 412M params (ViT-SO400M shape).
- **C-RADIOv4-H**: 631M params (ViT-H).

Both under NVIDIA Open Model License (commercial use permitted).

## Results

### Classification & segmentation (Table 1)

| Model | Zero-shot | kNN | ADE20k mIoU | VOC mIoU |
|---|---|---|---|---|
| RADIOv2.5-H | 82.51 | 85.81 | 51.58 | 85.97 |
| C-RADIOv3-H | 82.65 | 86.23 | 52.75 | 86.41 |
| **C-RADIOv4-SO400M** | 82.01 | 85.76 | 55.14 | **87.22** |
| **C-RADIOv4-H** | **83.09** | **86.59** | **55.20** | **87.24** |
| SigLIP2-g | 84.75 | 86.39 | 42.7 | 72.7 |
| DINOv3-7B | — | 85.42 | 55.9 | 86.6 |

C-RADIOv4-H matches DINOv3-7B on ADE20k (55.20 vs 55.9) and beats on VOC (87.24 vs 86.6) at ~10× fewer params.

### Resolution scaling (Table 2)

| Model | ADE20k @ 512px | @ 1024px | @ 1536px |
|---|---|---|---|
| DINOv3-7B | 55.9 | 57.3 | 57.8 |
| C-RADIOv4-H | 55.20 | 57.02 | 57.72 |

Strongly robust to over-resolution inference (trained up to 1152px, evaluated at 1536px).

### Probe3D (Table 4) — 3D awareness

| Model | Depth | Surface Normals | NAVI | SPair |
|---|---|---|---|---|
| RADIOv2.5-H | 85.69 | 62.46 | 60.89 | 56.24 |
| C-RADIOv3-H | 85.02 | 60.10 | 59.82 | 53.98 |
| C-RADIOv3-H+ | 86.18 | 62.52 | 62.10 | 58.54 |
| C-RADIOv4-SO400M | 85.29 | 61.91 | 62.44 | 60.01 |
| **C-RADIOv4-H** | 85.55 | 61.70 | **63.44** | **60.57** |

C-RADIOv4 is best-in-family on NAVI (cross-category correspondence) and SPair (keypoint correspondence).

### Instance segmentation via SAM3 replacement (Table 5, SA-Co/Gold cgF1)

| Model | metaclip_nps | sa1b_nps | crowded | fg_food | fg_sports_equipment | attributes | wiki_common | Avg |
|---|---|---|---|---|---|---|---|---|
| SAM3 | 47.3 | 53.7 | 61.1 | 53.4 | 65.5 | 54.9 | 42.5 | **54.1** |
| C-RADIOv4-H-VDT12 | 45.6 | 48.4 | 57.3 | 40.2 | 46.1 | 45.2 | 26.7 | 44.2 |
| C-RADIOv4-H-G | 45.9 | 48.8 | 57.4 | 40.9 | 46.5 | 45.9 | 27.3 | **44.7** |

Second-best model; inherits SAM3's uneven domain performance. Gap largest on domain-specific (`fg_sports_equipment`, `wiki_common`) — improving this is flagged as open research.

### Qualitative

- **Figure 1**: PCA feature visualization — C-RADIOv4-H has visibly cleaner object boundaries than C-RADIOv3-H.
- **Figure 2**: DINOv3 PCA shows out-of-place speckles (register-token and ViTDet-border artifacts); C-RADIOv4's adapter reproduces DINOv3's semantics *without the speckles*.
- **Figures 6–8**: SAM3 with RADIO encoder replacement matches SAM3 qualitatively on text/box prompts, and *fixes* SAM3's known "person"-query bug (SAM3 GitHub issue #253).

### Latency (Figures 5, 9)

ViTDet mode scales linearly in window count rather than quadratically in resolution. SO400M-VDT12 is faster than SAM3's ViT-L+ at same setting; H-VDT8 nearly as fast. At 4096px input, ViTDet window=16 is ~5× faster than full global attention for the Huge model.

## Why it matters

Three things shift with C-RADIOv4:

1. **Commercial unblock**: the first RADIO that can ship in a commercial product. Every bet in [[lifting-foundation-models-to-3d]] referencing RADIOv2.5 (LangSplat-on-RADIO, ConceptFusion+RADIO, Pipeline IX) was carrying an implicit NSCL-license risk that's now gone.
2. **Positional-noise hygiene**: the shift-equivariant loss is a principled fix to a mechanism-level pathology of multi-teacher distillation. The mechanism generalizes — anywhere a student distills from a ViT with register tokens or ViTDet artifacts (LangSplat's per-scene autoencoder, any RADIO-style follow-on), shift-equivariance should help.
3. **Any-resolution + ViTDet deployment**: the combination of stochastic-resolution training and ViTDet windowed inference makes C-RADIOv4 viable as a high-res vision encoder *and* a fast encoder at the same time — the resolution/latency trade-off that previously pushed users toward ToMe compression is partially dissolved.

## Pipeline contribution

- [[cradiov4-agglomerative-distillation_ranzinger2026]] (wrapper) — refined multi-resolution agglomerative distillation with the 2026 teacher set and four training-side upgrades. Candidate thread: [[open-vocab-2d-composition]] op:distilled-single-pass · stage: *unified frontend* · **SOTA swap**: RADIOv2.5 → C-RADIOv4 at the Pipeline C node. Expected gain: commercial-license unblock + matching DINOv3-7B on dense tasks at 10× fewer params + any-resolution support.
- [[shift-equivariant-distillation-loss_ranzinger2026]] (atomic, general-purpose) — random patch-aligned crop-shift between student and each teacher; loss evaluated on shared positions via $\mathcal{F}_{S \to T}$. Kills fixed-pattern positional-noise mimicry. Candidate thread: [[foundation-features-for-geometry]] op:default · stage: [[foundation-features.pretraining]] · mechanism travels to non-RADIO distillation (LangSplat autoencoder, Gaussian Grouping supervision, any ViT-to-ViT distillation).
- **Cross-thread transfer to [[lifting-foundation-models-to-3d]]**: C-RADIOv4 is now the preferred variant in Bets #014 (LangSplat+RADIO), #016 (ConceptFusion+RADIO), #020 (Pipeline IX). Commercial license resolved; cleaner per-Gaussian features from Fig 1 boundary improvement.
- **Cross-thread transfer to [[foundation-features-for-geometry]]**: C-RADIOv4 SOTA swap in the multi-task-frontend choice at stage 2 of op:default pipeline. +2.5 NAVI, +4 SPair over RADIOv2.5-H.
- **Largely realizes Bet #018** of [[open-vocab-2d-composition]]: "Distill SAM3 + DINOv3 into a RADIO-style student." C-RADIOv4 realizes the teacher set but does NOT carry SAM3's prompt-conditioning into the student — residual "prompt-conditioned student" remains open.

## Relation to prior work

- Directly refines [[heinrich2025_radiov25|RADIOv2.5]] (same family, iterative improvement).
- Builds on [[simeoni2025_dinov3|DINOv3]], [[carion2026_sam-3|SAM3]] (new teachers).
- PHI-S (Ranzinger 2024, no stub) — extended from spatial to summary features via angle-normalized loss.
- FeatSharp (Ranzinger 2025, no stub) — used as teacher-side upsampler for SigLIP2.
- ViTDet (Li et al. 2022, no stub) — brought back for efficient high-res inference.
- DAMP (Du et al. 2020, no stub) — multiplicative weight noise; minor.
- MESA (Du et al. 2022, no stub) — extended with shift-equivariance.
- Competes with [[simeoni2025_dinov3|DINOv3-7B]] at 10× fewer params on ADE20k and wins on VOC.
- Evidence-adjacent with [[heinrich2025_radiov25|RADIOv2.5]]'s `ToMe` — C-RADIOv4's any-resolution training *partially substitutes* for ToMe's fixed token budget (raw tokens at 1152px are already competitive without merging).

## Open questions / limitations

- Tech report, not peer-reviewed. No ablation isolating each of the four training-side changes individually — only full-system numbers are reported. Which of {stochastic-resolution, shift-equivariant loss, shift-equivariant MESA, angle-normalized summary loss} contributes most is not determined.
- 10-point cgF1 gap vs SAM3 on SA-Co/Gold: SAM3's vision encoder retains domain-specific signal (esp. `fg_sports_equipment`, `wiki_common`) that RADIO doesn't capture. The "RADIO replaces SAM3" claim is conditional on domain.
- Training cost is not reported. Stochastic-resolution with 10 sampled sizes and shift-equivariant loss with paired student+teacher crops both plausibly *increase* per-step compute — unclear whether total budget is net-neutral vs RADIOv2.5.
- Angular-dispersion normalization assumes teachers' angular distributions stay stable during training; may drift under aggressive augmentation.
- Capability frozen at distillation time (inherited from RADIOv2.5): a 2027 SigLIP3 / DINOv4 / SAM4 triggers a full retrain, not a partial upgrade. The decomposed-pipeline Trident pattern retains this modularity advantage (see [[open-vocab-2d-composition]] trade-off discussion).

## Code & license

**License upgrade** over RADIOv2.5 (which was NSCL, non-commercial). C-RADIOv4 weights and code ship under the NVIDIA Open Model License, which permits commercial use with standard terms (attribution, responsible-use compliance, no exploitation for competing foundation-model training). The SAM3-replacement variant lives at a separate fork: [github.com/mranzinger/sam3-radio](https://github.com/mranzinger/sam3-radio) — same license.

## References added to the wiki

- [[cradiov4-agglomerative-distillation_ranzinger2026]] (new idea, refines predecessor)
- [[shift-equivariant-distillation-loss_ranzinger2026]] (new atomic idea)
- [[radiov25-agglomerative-distillation_heinrich2025]] (updated with `refined_by:` edge)
- [[siglip2]] (stub — load-bearing teacher)
- [[phi-s]] (stub — load-bearing baseline)
- [[featsharp]] (stub — load-bearing teacher-upsampling method)
- [[simeoni2025_dinov3]] (cross-link updated)
- [[carion2026_sam-3]] (cross-link updated)

## In the wiki

- Task-by-task recipes for using C-RADIOv4's outputs to produce segmentation masks (instance + semantic + open-vocab) and depth maps: [[radio-dense-prediction]].
- Part of the [[foundation-model]] family (agglomerative distillation lineage: AM-RADIO → RADIOv2.5 → C-RADIOv3 → C-RADIOv4).
- Referenced by [[open-vocab-2d-composition]] op:distilled-single-pass (SOTA filler as of 2026-04-22).
- Referenced by [[foundation-features-for-geometry]] op:default as the multi-task-frontend backbone choice.
- Referenced by [[lifting-foundation-models-to-3d]] as the preferred agglomerative-distillation backbone in Bets #014, #016, #020.
