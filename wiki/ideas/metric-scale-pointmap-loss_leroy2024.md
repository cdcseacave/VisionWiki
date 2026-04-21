---
title: Metric-scale pointmap regression loss (disable scale normalization on metric GT)
type: idea
source_paper: wiki/papers/leroy2024_mast3r.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.geometry-output]
collapses: []
splits_into: []
rewrites: {}

inputs: [pointmap-prediction-per-pixel, ground-truth-pointmap, metric-gt-flag]
outputs: [metric-scale-pointmap-prediction]
assumptions: [training-mix-contains-metric-gt-subset, camera-pair-reconstruction]
requires_upstream_property: [dust3r-style-per-pixel-pointmap-output]
requires_downstream_property: [consumer-can-use-metric-pointmap]
learned_params: [no-new-params-loss-change-only]
failure_modes: [metric-overfitting-to-metric-subset-domain, ambiguity-when-mixed-with-synthetic-or-up-to-scale-data, regressed-scale-degrades-outside-training-regime]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mast3r, pointmap, metric-scale, loss-function, regression, feed-forward]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

[[dust3r|DUSt3R]]'s pointmap regression loss (Eq. 6 of [[leroy2024_mast3r]]) normalizes predicted and ground-truth pointmaps by their mean distance to origin:

$$\ell_{\text{regr}}(v, i) = \left\| \frac{1}{z}X_i^{v,1} - \frac{1}{\hat{z}}\hat{X}_i^{v,1} \right\|, \quad z = \frac{1}{|\mathcal{V}|}\sum_{i \in \mathcal{V}}\|X_i\|, \quad \hat{z} = \frac{1}{|\hat{\mathcal{V}}|}\sum_{i \in \hat{\mathcal{V}}}\|\hat{X}_i\|$$

This makes regression *scale-invariant*: only shape matters, absolute depth does not. MASt3R makes a one-line change: when the ground-truth pointmap is metric, **set `z := ẑ`**. The loss becomes `||X − X̂||/ẑ`, so the network is now penalized if its predictions drift in *absolute scale* from the metric ground truth. The model has no new parameters; it only sees a different gradient.

Concretely, the MASt3R training mix contains 14 datasets — 10 of which provide metric ground-truth (Habitat, ARKitScenes, BlendedMVS, MegaDepth, Static Scenes 3D, ScanNet++, CO3D-v2, Waymo, MapFree, WildRGB-D, VirtualKitti, Unreal4K, TartanAir, internal). For these, the flag is set; for the rest (e.g., MegaDepth portions derived from SfM), the DUSt3R scale-invariant loss applies. The confidence-weighted aggregation and InfoNCE loss are unchanged.

The mechanism is more subtle than "train for metric": the network must learn a *feature-to-absolute-distance* mapping purely from pixel data, without any hand-coded camera intrinsics, focal, or depth prior. Because metric GT is interleaved batch-by-batch with scale-normalized GT, the same backbone simultaneously develops scale-aware predictions (when asked) and scale-free predictions (when scale is ambiguous).

## Why it wins

**Isolating ablation — Table 1, Row IV vs V (Map-free validation):**

| Row | match | depth source | VCRE AUC | Pose AUC | Med. trans err. |
|-----|-------|--------------|----------|----------|------------------|
| IV (MASt3R feat + DPT depth) | feat | DPT-KITTI | 0.752 | 0.435 | 0.93m |
| V (MASt3R feat + MASt3R auto depth) | feat | MASt3R metric | **0.934** | **0.746** | **0.46m** |

Swapping the external DPT depth for MASt3R's own metric pointmap prediction is the largest single ablation jump in the paper. VCRE Precision rises from 51.5% to 75.9%.

**Test-set generalization — Table 2:**
- MASt3R + DPT depth: 0.726 VCRE AUC.
- MASt3R + auto depth: **0.933** VCRE AUC.
- MASt3R direct regression (no matching at all, PnP on MASt3R's pointmap X^{2,1} with the full 2M pixel-to-3D set): **0.941** VCRE AUC — beats even the matching-based variant.

The direct-regression result is the strongest evidence that metric pointmaps are *load-bearing* for feed-forward localization: given a high-quality metric pointmap, the network needs neither correspondences nor RANSAC to produce state-of-the-art pose — a single forward pass from a pair of raw images gives a 30-36cm median translation error on an extremely challenging map-free benchmark. This is distinct from DUSt3R's up-to-scale output, which requires an external depth alignment step to become metric and loses the first-pass win.

Closed-form causal story: *up-to-scale* pointmap + *external* metric depth source compounds two different noise processes (pointmap scale ambiguity + monocular depth calibration noise). Metric pointmap output collapses the two sources into one learned process trained end-to-end on the task that matters.

## Preconditions & compatibility

- **Upstream**: any DUSt3R-family pointmap head.
- **Training**: ≥1 metric-GT dataset in the training mix. Without metric data, the modified loss is equivalent to DUSt3R's original.
- **Downstream**: any consumer that benefits from absolute scale. In [[leroy2024_mast3r]] this includes Map-free localization (metric translation error is the headline), MVS triangulation, and PnP-based direct pose regression.
- **Compatible with** [[dust3r-matching-head_leroy2024]] (independent axis). A pointmap-only DUSt3R trained with this loss already wins at metric localization; the matching head is additive.
- **Not compatible with** training regimes that mix purely up-to-scale data without per-sample flags — would destabilize training.

## Trade-offs vs. the decomposed pipeline

Not applicable — `drop-in` scope. The modification adds a capability (metric output) without removing anything; the up-to-scale behavior survives on non-metric training samples.

## Open questions

- **Domain transfer**: the 10 metric datasets span indoor + outdoor + synthetic. How well does metric prediction generalize to a new metric domain (e.g., aerial, underwater)? Paper doesn't test this.
- **Metric prediction from monocular input**: MASt3R receives a pair; could the same loss recover metric depth from a single image via a single-image DUSt3R variant? Not addressed.
- **Interaction with [[dust3r-matching-head_leroy2024]]**: if matching is already strong, does metric-scale output provide diminishing returns? Table 2 "direct regression" row suggests otherwise — 94.1 AUC without matching at all — but per-dataset ablation isn't provided.
- **Reliance on GT depth quality**: MegaDepth's SfM-derived depth is only approximately metric (scale set by COLMAP reconstruction). Training on noisy-metric labels hasn't been ablated against training on only high-quality metric labels (ARKitScenes, TartanAir).
- **Regression-only localization risks**: Table 2's direct-regression variant is tested only in calibrated settings. When GT intrinsics are wrong or unknown, the direct-regression pose *degrades sharply* (Table 4 "MASt3R direct reg. top1" column — 1.5/4.5/60.7 on AachenDayNight day vs 79.6/93.5/98.7 for matching-based). Metric pointmap output is robust; direct regression from it is brittle to calibration mismatch.
