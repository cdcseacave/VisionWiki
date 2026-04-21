---
title: Fast reciprocal nearest-neighbor matching via iterative sub-sampling
type: idea
source_paper: wiki/papers/leroy2024_mast3r.md
also_in: []

scope: drop-in
stages: [feature-matching.reciprocal-matching]
collapses: []
splits_into: []
rewrites: {}

inputs: [dense-feature-map-pair-D1-D2, unit-norm-descriptors, budget-k]
outputs: [reciprocal-correspondence-set-bounded-by-k]
assumptions: [cosine-similarity-metric, unit-norm-descriptors, pair-descriptors-aligned-in-shared-metric-space]
requires_upstream_property: [pixel-aligned-dense-descriptor-map]
requires_downstream_property: [consumer-accepts-bounded-correspondence-set]
learned_params: []
failure_modes: [k-too-small-misses-low-saliency-correspondences, strongly-repetitive-scenes-converge-to-wrong-cycles, grid-initialization-biased-toward-uniform-sampling]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [matching, reciprocal-nn, faiss, iterative, outlier-filtering, mast3r]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Given dense per-pixel descriptor maps `D¹, D² ∈ ℝ^{H×W×d}`, the classical reciprocal-match set

$$\mathcal{M} = \{(i,j) \mid j = \mathrm{NN}_2(D_i^1) \land i = \mathrm{NN}_1(D_j^2)\}$$

costs `O(W²H²)` pairwise comparisons — prohibitive for dense feature maps of modern matchers. K-d trees help in low dimension but collapse for `d = 24` (curse of dimensionality). The paper proposes a **sub-sampled iterative cycle-finder**:

1. Initialize `U⁰ = {u₁⁰, …, u_k⁰}` — a regular grid of `k ≪ WH` pixels in image 1.
2. For `t = 0, 1, 2, …`:
   - Forward step: `V^t = [NN₂(D¹_u)]_{u ∈ U^t}`.
   - Backward step: `U^{t+1} = [NN₁(D²_v)]_{v ∈ V^t}`.
   - Collect cycle fixed points: `M^t_k = {(U^t_n, V^t_n) : U^t_n = U^{t+1}_n}`.
   - Drop converged pixels: `U^{t+1} ← U^{t+1} \ U^t`.
3. Terminate after ≤6 iterations (Fig. 3 center: `U^t` drops to near zero by `t = 5`).
4. Output: `M_k = ⋃_t M^t_k`. `|M_k| ≤ k`.

The forward/backward steps are a single FAISS index lookup in both images, `O(k·d·log(WH))` total per iteration. Total cost `O(k·WH)` (FAISS index build dominates when `k` is small). Compared to the naive `O(W²H²)`, speedup is `WH / k`.

Two features are non-obvious:
- **Sub-set, not approximation**: `M_k ⊆ M` is provably a subset of the true reciprocal set, not an approximate one. Convergence guarantees in the supplementary.
- **Implicit outlier filtering**: only pixels whose forward-then-backward cycle closes *repeatedly* survive. An outlier descriptor that happens to have some NN in image 2 is discarded when the backward pass routes away from it.

## Why it wins

The load-bearing ablation is Figure 3 right of [[leroy2024_mast3r]] on Map-free (VCRE AUC < 90px, CPU runtime):

| Subsampling k | Runtime (CPU, single core) | VCRE AUC |
|---------------|----------------------------|----------|
| all (no subsampling) | ~100 s | ~0.72 |
| k = 49000 | ~30 s | ~0.73 |
| k = 12000 | ~3 s | ~0.74 |
| k = 3000 | ~1 s (64× faster than `all`) | **~0.75** |
| k = 768 | ~0.3 s | ~0.73 |

**Performance monotonically improves as `k` drops** from `all` to `3000`, then plateaus/degrades. Paper notes this is "surprising" but the explanation is mechanistic: each cycle-consistency check is an independent outlier test, and smaller `k` yields a more rigorous filter (fewer chances for spurious cycles). Below `k = 768`, too-sparse sampling misses legitimate correspondences and recall drops.

The 64× speedup at `k = 3000` over full-resolution reciprocal matching is what makes MASt3R's dense matching *practical* for inference at all — the decoder can run in milliseconds, but naive reciprocal-NN would spend 2+ seconds per image pair. This is also what motivates [[iterative-ray-pointmap-matching_murai2025]] in MASt3R-SLAM's real-time budget (that paper uses a different inner loop but the same reciprocal-subsampling principle).

## Preconditions & compatibility

- **Upstream descriptor space**: unit-norm descriptors + cosine (or Euclidean equivalent) similarity. Paper uses L2-normalized 24-dim vectors from [[dust3r-matching-head_leroy2024]]. Works with any similar dense matcher ([[edstedt2025_roma-v2|RoMa v2]] coarse features, LoFTR coarse features, DINO-family patch features).
- **Independent of architecture**: the algorithm doesn't care where the descriptors come from; it only requires a pair of aligned dense descriptor maps.
- **Compatible with** [[coarse-to-fine-window-covering-matching_leroy2024]] (the outer loop calls this inner matcher per window pair).
- **Not compatible with** matchers that output already-sparse matches (LightGlue, SuperGlue): the algorithm needs a dense descriptor field to sample from.

## Trade-offs vs. the decomposed pipeline

Not applicable — `drop-in` scope. The algorithm replaces a monolithic `O(W²H²)` all-pairs reciprocal matcher with a bounded-output `O(k·WH)` variant; the "decomposed pipeline" would be some hypothetical staged reciprocal matcher, which the paper doesn't break into sub-stages.

## Open questions

- **k selection as a function of scene / matcher**: the paper fixes `k = 3000` for Map-free. Is the optimum scene-dependent? Matcher-dependent? Untested on e.g. highly repetitive textures where cycle-consistency is weakest.
- **Alternate initialization strategies**: regular grid is the simplest choice. Saliency-based initialization (focus on high-confidence descriptor regions) might do better with smaller `k`.
- **Convergence count**: `≤6` iterations is empirical on Map-free. Theoretical bounds in the supplementary rely on descriptor-space properties that may not hold on all datasets.
- **GPU implementation**: all timings are single-core CPU. The inner loop is embarrassingly parallel in `k`; a GPU implementation would dominate every baseline.
- **Composition with learned verification**: could a learned outlier-scoring head on top of `M_k` further improve accuracy? The algorithm's implicit filter is non-learned; a learned one might compose.
