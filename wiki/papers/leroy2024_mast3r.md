---
title: "Grounding Image Matching in 3D with MASt3R"
type: paper
tags: [feature-matching, 3d-reconstruction, feed-forward, dust3r, mast3r, metric-depth, pose-estimation, infonce, reciprocal-matching]
created: 2026-04-21
updated: 2026-04-21
sources: []
local_paper: papers/feature-matching/leroy_2024_mast3r.pdf
url: https://arxiv.org/abs/2406.09756
code: https://github.com/naver/mast3r
license_paper: arxiv-nonexclusive
license_code: CC-BY-NC-SA-4.0
status: draft
---

📄 [Full paper](../../papers/feature-matching/leroy_2024_mast3r.pdf) · [arXiv](https://arxiv.org/abs/2406.09756) · [code](https://github.com/naver/mast3r)

_Paper license: `arxiv-nonexclusive` · Code license: `CC-BY-NC-SA-4.0`_

## TL;DR

MASt3R ("Matching And Stereo 3D Reconstruction") augments [[dust3r|DUSt3R]] with a second prediction head that outputs dense local feature descriptors, jointly trained with InfoNCE. The resulting model is a single feed-forward network that produces metric-scale 3D pointmaps *and* pixel-precise dense correspondences — unifying stereo reconstruction, camera calibration, and feature matching. A new iterative reciprocal-NN algorithm is 64× faster than full dense reciprocal matching while simultaneously being more accurate, and a greedy window-pair set-cover extends matching to high-resolution images despite the ViT-512 training cap. On Map-free localization (the hardest viewpoint-change benchmark), MASt3R beats the best published method (LoFTR+KBR) by 30 absolute AUC points.

## Problem

Image matching underpins every 3D vision pipeline — SfM, visual localization, SLAM — yet is almost always formulated as a **2D** problem: find pixel `i` in image 1 that corresponds to pixel `j` in image 2 by comparing local descriptors. This framing is lossy: two pixels correspond iff they observe the same 3D point, so a 2D-only formulation discards the exact structure the downstream pipeline needs to recover.

State-of-the-art 2D matchers — LoFTR, SuperGlue+SuperPoint, Kapture+R2D2 — achieve high precision on near-duplicates but degrade steeply under large viewpoint change. LoFTR gets only **34% VCRE AUC** on Map-free localization (the benchmark specifically designed to stress viewpoint invariance). Conversely, [[dust3r|DUSt3R]] — a pointmap-regression network that *does* reason in 3D — is robust to extreme viewpoints but produces imprecise pixel correspondences (median rotation error ~7° vs ~3° for matching-based approaches at best), because regression inherently rewards "close enough" rather than exact pixels.

The authors' thesis: matching *is* 3D. The network that produces matches should be 3D-aware. But 3D regression alone gives imprecise pixels, so the matching supervision is needed on top. Fixing both sides of this trade-off is the paper's goal.

## Method

**Architecture.** MASt3R inherits the [[dust3r|DUSt3R]] backbone: two images `I¹, I²` pass through a shared ViT-L encoder `H^v = Encoder(I^v)`, then through two intertwined transformer decoders that cross-attend between views producing `H'^v = Decoder(H¹, H²)`. Two **3D heads** `Head^v_{3D}` regress per-pixel pointmaps `X^{v,1} ∈ ℝ^{H×W×3}` in image-1 coordinates plus confidence `C^v`. This is identical to DUSt3R.

**New matching heads.** Two `Head^v_{desc}` — each a 2-layer MLP with GELU activation — predict dense local feature maps `D^v ∈ ℝ^{H×W×d}` with `d = 24`. Descriptors are L2-normalized.

**Matching loss.** Ground-truth 3D correspondences `Ĉ = {(i, j) : X̂_i^{1,1} = X̂_j^{2,1}}` come from reciprocal NN on GT pointmaps. InfoNCE cross-entropy:

$$\mathcal{L}_{\text{match}} = -\sum_{(i,j) \in \hat{C}} \log \frac{s_\tau(i,j)}{\sum_{k \in \mathcal{P}^1} s_\tau(k,j)} + \log \frac{s_\tau(i,j)}{\sum_{k \in \mathcal{P}^2} s_\tau(i,k)}, \quad s_\tau(i,j) = \exp(-\tau D_i^{1\top} D_j^2)$$

Total: `L_total = L_conf + β · L_match`, β=1. See [[dust3r-matching-head_leroy2024]].

**Metric-scale loss variant.** DUSt3R's regression loss normalizes by mean distance-to-origin (scale-invariance). MASt3R sets `z := ẑ` on the metric-GT subset of training data (10 of 14 datasets), forcing the network to regress absolute metric scale. See [[metric-scale-pointmap-loss_leroy2024]].

**Fast reciprocal matching.** Naive reciprocal-NN on dense descriptors is `O(W²H²)`. FAISS helps but is still `O(WH · log(WH))` per image. The paper introduces an iterative sub-sampling algorithm that finds cycles `u → NN₂(D¹_u) → NN₁(D²_·) = u` starting from a `k`-pixel grid, converges in ~5 iterations, outputs `|M_k| ≤ k` matches. Subsampling is both faster *and* more accurate due to implicit outlier filtering. See [[fast-reciprocal-nn-matching_leroy2024]].

**Coarse-to-fine for high-res inputs.** ViT trained at 512px can't handle 1600×1200 test images. The solution is a coarse matching pass at 512px, then greedy window-pair set-cover of ≥90% of coarse matches on full-resolution crops, then per-window matching with principal-point-preserving homographies. See [[coarse-to-fine-window-covering-matching_leroy2024]].

**Training.** 14-dataset mixture (Habitat, ARKitScenes, BlendedMVS, MegaDepth, Static Scenes 3D, ScanNet++, CO3D-v2, Waymo, Map-free, WildRGB-D, VirtualKitti, Unreal4K, TartanAir, internal NAVER LABS dataset). 10 provide metric GT; all used for match supervision. Model initialized from public DUSt3R checkpoint. 35 epochs, 650k pairs/epoch, cosine LR schedule, `lr₀ = 1e-4`, `d = 24`, β = 1, aggressive random-crop augmentation (crops transformed by homography to preserve the central principal point).

## Results

**Map-free localization** (Table 2 test set, VCRE AUC & Pose Error, single query image from extreme viewpoint):
| Method | VCRE AUC | Pose AUC | Median trans (m) |
|--------|----------|----------|-------------------|
| SIFT + DPT | 0.504 | 0.252 | 2.93 |
| SuperPoint+SuperGlue + DPT | 0.602 | 0.346 | 1.88 |
| [[sun2021_loftr|LoFTR]] + KBR | 0.634 | 0.295 | 2.23 |
| [[dust3r|DUSt3R]] + DPT | 0.697 | 0.394 | 0.97 |
| **MASt3R + DPT** | 0.726 | 0.456 | 0.80 |
| **MASt3R + MASt3R (auto)** | **0.933** | **0.740** | **0.36** |
| **MASt3R direct regression (no match)** | **0.941** | **0.777** | 0.42 |

Beats LoFTR+KBR (the best previously-published method) by **+30 VCRE AUC absolute**. The direct-regression variant — single forward pass, no RANSAC, no matching — is the best configuration on this benchmark.

**Relative pose on CO3Dv2 / RealEstate10K** (Table 3): MASt3R improves mAA(30) over DUSt3R by +15.2 on RealEstate10K (76.4 vs 61.2) pairwise. Top multi-view method on both datasets.

**Visual localization on Aachen Day-Night / InLoc** (Table 4): MASt3R is competitive with specialized matchers (SuperPoint+LightGlue, Kapture+R2D2) on Aachen; on InLoc DUC2 it achieves **71.0/87.0/91.6** day, matching or exceeding the top specialized methods despite being trained for a different objective.

**Dense MVS on DTU zero-shot** (Table 3e): MASt3R's overall Chamfer is **0.374mm**, vs DUSt3R's **1.741mm** (4.6× improvement), outperforming all zero-shot baselines.

## Why it matters

MASt3R establishes that the right formulation of dense feature matching is as a by-product of 3D-aware regression, trained jointly. This observation:

1. **Consolidates three previously-separate pipelines** — matching, calibration, pose estimation, and now implicitly reconstruction — into a single feed-forward pass.
2. **Shifts the SOTA for extreme-viewpoint matching** from 2D attention architectures (LoFTR and descendants) to 3D-grounded ones. The 30-point absolute gain on Map-free is the single largest single-paper jump on that benchmark.
3. **Makes metric-scale pose attainable from a single forward pass** — the direct-regression variant (94.1 VCRE AUC) bypasses matching altogether when the pointmap is good enough, changing what a "feed-forward SfM" can mean.
4. **Becomes the backbone of a research line** — [[murai2025_mast3r-slam|MASt3R-SLAM]], MASt3R-SfM (referenced in this paper), [[jang2025_pow3r|PoW3R]], [[pataki2025_mp-sfm|MP-SfM]]'s matcher option, [[jin2026_zipmap|ZipMap]], [[chen2026_ttt3r|TTT3R]] all descend from or plug into this architecture. Fits into the Tier-3 node of the [[feed-forward-structure-from-motion]] thread's taxonomy.

Also relevant to the [[feed-forward-structure-from-motion]] capability gap *"calibration-grade outdoor accuracy from fully feed-forward methods"* — Map-free's direct-regression result is the strongest existing evidence that this gap is closable on Tier 3 alone (no BA tail required).

## Pipeline contribution

Four first-class ideas extracted in Step 3 of this ingest:

- [[dust3r-matching-head_leroy2024]] · candidate thread: [[feed-forward-structure-from-motion]] · mechanism: dual-head DUSt3R with a 2-layer InfoNCE-trained descriptor head; 3D regression and matching mutually regularize, validated by bidirectional Table 1 ablations · expected gain: pixel-precise matching on extreme-viewpoint pairs (median rot 3.0° vs DUSt3R 9.4°).
- [[metric-scale-pointmap-loss_leroy2024]] · candidate thread: [[feed-forward-structure-from-motion]] · mechanism: drop the `1/z` normalization on metric-GT training batches so the network regresses absolute scale directly; no new parameters · expected gain: enables single-forward-pass metric-pose localization (94.1 AUC on Map-free from pointmap alone, beating all matching-based variants).
- [[fast-reciprocal-nn-matching_leroy2024]] · candidate thread: cross-cutting (any dense-matcher thread) · mechanism: iterative sub-sampled reciprocal-NN with cycle verification; provably a subset of the full reciprocal set with implicit outlier filtering · expected gain: 64× CPU speedup + AUC *improvement* (0.72 → 0.75 Map-free) over naive all-pairs reciprocal matching.
- [[coarse-to-fine-window-covering-matching_leroy2024]] · candidate thread: [[feed-forward-structure-from-motion]] · mechanism: greedy window-pair set-cover at full resolution seeded by a 512px coarse pass; uses homography crops to preserve principal point · expected gain: enables sub-pixel matching at arbitrary input resolution despite a 512px-bounded matcher; 4.6× Chamfer reduction on DTU zero-shot over DUSt3R.

Mechanism-level details live on the idea pages; this section is cross-reference only.

## Relation to prior work

- **Direct extension of** [[dust3r|DUSt3R]] (Wang et al. CVPR 2024): same architecture, initialized from DUSt3R checkpoint, adds the descriptor head and metric-scale loss variant.
- **Pretraining lineage**: DUSt3R inherits from CroCo (masked cross-view reconstruction). MASt3R keeps this pipeline.
- **Contrasts** [[sun2021_loftr|LoFTR]] and SuperGlue/[[lightglue]] — all 2D matchers. MASt3R's 30-point Map-free gain over LoFTR+KBR is the headline refutation of the 2D formulation for extreme viewpoints.
- **Contrasts** RayDiffusion (diffusion over ray bundles): 3D-aware pose but no matching; MASt3R on CO3Dv2 pairwise beats RayDiffusion by **+0.3 RTA@15 / +5.1 RRA@15**, and produces explicit correspondences as a bonus.
- **Consumes** metric-trained monocular depth predictors (DPT-KITTI, [[wang2025_moge|MoGe]]) indirectly — Table 1 shows MASt3R's auto-depth dominates external depth sources at the pose task.
- **Used downstream by** [[murai2025_mast3r-slam|MASt3R-SLAM]] (real-time dense SLAM on MASt3R priors), [[pataki2025_mp-sfm|MP-SfM]] (optional matcher), [[jang2025_pow3r|PoW3R]] (conditional DUSt3R/MASt3R extension), [[edstedt2025_roma-v2|RoMa v2]] comparisons, and many others in the DUSt3R family.

## Open questions / limitations

**Author-stated**: (i) MASt3R trains only on pinhole images, degrades with distortion. (ii) 512px max training resolution fundamentally caps the architecture; workarounds (coarse-to-fine, larger ViTs) don't fix the training recipe. (iii) Scale inconsistency across independently-predicted pairs — metric pointmaps for different pair choices can disagree in absolute scale in edge cases, requiring Sim(3) alignment downstream.

**Analyst skepticism**:
- The `d = 24` descriptor dimension is tiny for a matching descriptor (SIFT: 128, SuperPoint: 256, RoMa: 512+). Why does `d = 24` work? No ablation on descriptor dimension is provided. Possibly because 3D regression supervision shapes descriptor space into a metric that doesn't need high dimensionality — but this is speculative. Would like to see descriptor-dim × matching-head ablation.
- Direct-regression pose (Table 4 MASt3R direct reg. top1) collapses on Aachen (1.5/4.5/60.7) despite 94.1 AUC on Map-free. The contrast suggests direct regression is brittle to calibration / scale-mismatch between query and reference — not a generally-deployable alternative to matching.
- Fig 3 (right) shows performance *improving* as subsampling tightens, monotonically. Past `k ≈ 3000` it doesn't drop until `k < 768`. This is a very flat optimum — suspicious. Either (a) the dataset's "outliers" are easy enough that any filtering helps, or (b) the matching evaluator (VCRE with 90px threshold) is not fine-grained enough to distinguish 3000 vs 49000. Not resolved in the main paper; supplementary's convergence analysis may clarify.
- No standalone ablation of `coarse-to-fine` on Map-free or Aachen — it's used for MVS but not isolated. The idea is adopted on faith for high-resolution downstream tasks.
- Training mix includes an *internal NAVER LABS dataset* of unreported size — reproducibility caveat; community ports would start with one fewer dataset.

## Code & license

**Code**: [github.com/naver/mast3r](https://github.com/naver/mast3r) — CC-BY-NC-SA-4.0 (non-commercial, share-alike). **Model checkpoints**: same license on the weights *plus* each training dataset's own license applies; per the repo's `CHECKPOINTS_NOTICE`, the Map-free dataset license is particularly restrictive. Result: **not usable for commercial products as-is, even via the released weights**. Retraining on a commercial-safe data mix is the only path for downstream commercial deployment.

Per §6.15, this license constraint is informational: does not block bet adoption, confidence, or SOTA composition. It matters only at implementation time for commercial targets.

## References added to the wiki

- [[dust3r-matching-head_leroy2024]]
- [[metric-scale-pointmap-loss_leroy2024]]
- [[fast-reciprocal-nn-matching_leroy2024]]
- [[coarse-to-fine-window-covering-matching_leroy2024]]
- [[feature-matching.reciprocal-matching]] (new stage)

Updated:
- [[mast3r|MASt3R]] (method page: stub → draft with mechanism detail)
- [[dust3r|DUSt3R]] (method page: successor note)
- [[feed-forward-structure-from-motion]] (primary thread: MASt3R now a first-class Tier-3 entry, new Pass B bet)
- [[relative-pose-estimation]] (thread: formalize MASt3R references with idea wikilinks; new candidate for direct-regression pose)
