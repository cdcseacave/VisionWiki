---
title: Multi-view transformer feature-track refinement with reference-location search
type: idea
source_paper: wiki/papers/he2023_detector-free-sfm.md
also_in: []

scope: stage-swap
stages: [sfm.feature-track-refinement]
collapses: []
splits_into: []
rewrites: {}

inputs: [coarse-feature-tracks, multi-view-image-patches, current-coarse-poses]
outputs: [refined-subpixel-tracks, per-view-heatmap-variance]
assumptions: [dense-matcher-provides-coarse-tracks, coarse-pose-exists-for-reference-view-scale-ranking, static-scene]
requires_upstream_property: [coarse-tracks-from-quantized-matches, per-track-minimum-2-views]
requires_downstream_property: [geometric-BA-accepts-weighted-reprojection]
learned_params: [multi-view-feature-transformer-2-group-self-cross-attention, s2dnet-cnn-backbone]
failure_modes: [symmetric-scenes-minimum-variance-picks-wrong-reference, heavy-occlusion-degrades-ref-view-scale-estimate, p-or-w-mis-sized-for-matcher-stride]

requires: [coarse-to-fine-detector-free-sfm-bridge_he2023]
unlocks: [iterative-ba-plus-track-topology-adjustment_he2023]
co_requires: [coarse-to-fine-detector-free-sfm-bridge_he2023, iterative-ba-plus-track-topology-adjustment_he2023]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, track-refinement, transformer, attention, pixsfm-alternative, detector-free-sfm]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
Given a feature track `T_j = {x_k ∈ ℝ² | k=1..N_j}` from the coarse SfM output, pick a **reference view** `r` and treat the other `N_j - 1` views as **queries**. Extract `p×p` (p=15) CNN patches from S2DNet centered on each view's current keypoint. Feed all patches through a multi-view feature transformer: 2 groups of (self-attention across flattened `p²×p²` patch tokens within a view, cross-attention between reference and query views). Output: multi-view-consistent transformed features `F_k ∈ ℝ^{p×p×c}`.

Correlation: fix a query point in the reference view's feature map, correlate it with each query view's transformed features → per-query `p×p` heatmap = distribution over refined keypoint positions. Softmax to probability, take expectation = refined location `x_k*`, variance = per-view uncertainty.

Reference-location search: the reference point is not fixed to the coarse keypoint. Instead, sample a `w×w` (w=7) grid of candidate reference points around it, run the correlation for each, form a candidate track per sample. **Select the candidate track whose summed per-view variance is minimum** → this is the refined track `T_j*`.

Reference-view selection rule: pick the view whose current depth-value rank (from the coarse model) is the median across the track — this minimizes scale difference between reference and queries, improving matchability. Training loss is L2 between predicted refined keypoints and ground-truth tracks from MegaDepth.

Crucial design choice vs. PixSfM: no feature-metric BA. Once tracks are refined in this one forward pass, geometric BA (reprojection-only) is sufficient. Feature patches are never kept in memory for BA, which is why memory is 3 orders of magnitude smaller (Table 4 of [[he2023_detector-free-sfm]]).

## Why it wins
Causal story: attention-transformed features encode multi-view context (occluded regions, foreshortening) that plain CNN features miss, so the correlation is more discriminative and its minimum is sharper (Fig. 7 shows heatmap sharpening after transformer). Reference-location search is a soft relaxation of keypoint position in the reference view — without it the reference keypoint itself is frozen at the quantized coarse location.

Isolating ablations (Table 3 (4), ETH3D 1cm/2cm):
- Full model: 80.38 / 89.01.
- w/o transformer: 71.85 / 82.66 (−8.5 / −6.4).
- w/o ref. location search: 76.66 / 86.79 (−3.7 / −2.2).
- w/o topology adjustment (next idea's ablation): 75.78 / 85.47 (−4.6 / −3.5).

Transformer is the largest contributor; ref-location search and topology adjustment each add independently.

## Preconditions & compatibility
- **Upstream**: needs coarse tracks. Provided by [[coarse-to-fine-detector-free-sfm-bridge_he2023]]; works with any coarse SfM (detector-based or detector-free) that produces multi-view tracks.
- **Downstream**: outputs refined tracks with per-view uncertainty, consumed by any geometric BA. If the downstream BA is inverse-variance-weighted, uncertainty propagates; otherwise uncertainty is discarded without harm.
- **Hardware**: transformer runs on GPU; the reported config uses 4× NVIDIA V100 for parallelized matching, 16 CPU cores for BA. Tight GPU-CPU coupling is a deployment consideration.
- **Model**: MegaDepth-trained checkpoint; generalization to indoor / aerial / moon-surface reportedly holds but is not exhaustively ablated.

## Trade-offs vs. the decomposed pipeline
Not applicable — `stage-swap` scope. The swap preserves the `sfm.feature-track-refinement` interface; no pipeline shape change.

## Open questions
- **Plug into InstantSfM**: [[zhong2026_instantsfm|InstantSfM]] has GPU-native BA but no track-refinement stage beyond the COLMAP default. Does plugging this refiner into InstantSfM give the memory efficiency of both? Bet candidate in [[gpu-native-sfm]].
- **Foundation-feature backbone**: S2DNet is a 2020 CNN. Replacing with frozen DINOv3 features (no retraining needed per [[edstedt2025_roma-v2|RoMa v2]]'s recipe) is an obvious next step. Feeds [[foundation-features-for-geometry]].
- **Scaling**: `w=7` reference-location search multiplies compute by 49. For 10K-image sequences, is this refinement-time bottleneck the next target?
