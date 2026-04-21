---
title: Coarse-to-fine matching via greedy window-pair set-cover
type: idea
source_paper: wiki/papers/leroy2024_mast3r.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.frame-matching]
collapses: []
splits_into: []
rewrites: {}

inputs: [image-pair-at-arbitrary-resolution, base-matcher-with-fixed-512px-budget]
outputs: [full-resolution-correspondence-set]
assumptions: [base-matcher-handles-512px-crops, principal-point-preserved-under-crop, coarse-match-set-covers-overlap]
requires_upstream_property: [base-matcher-outputs-correspondences-at-crop-resolution]
requires_downstream_property: [consumer-accepts-full-resolution-correspondences]
learned_params: []
failure_modes: [coarse-pass-misses-high-frequency-only-matches, 90-percent-coverage-threshold-too-greedy-for-sparse-texture, homography-principal-point-correction-fails-near-image-edges]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [matching, coarse-to-fine, high-resolution, set-cover, mast3r, resolution-bridge]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

ViT-based matchers are capacity-bounded: [[mast3r|MASt3R]] trains at ≤512px max dimension; [[dust3r|DUSt3R]] same. Larger test-time inputs degrade performance because attention generalizes poorly to unseen token counts. Standard workaround: downsample to 512px, match, upsample correspondences back — loses sub-pixel accuracy and fine-detail matches.

The paper's fix is a **greedy set-cover over high-resolution window pairs**:

1. **Coarse pass**: downsample images to ≤512px, run MASt3R + [[fast-reciprocal-nn-matching_leroy2024]] → coarse correspondences `M⁰_k`.
2. **Window tiling**: generate a grid of 512×512 window crops `W¹, W² ∈ ℝ^{w×4}` on each *full-resolution* image, adjacent crops overlap by 50%.
3. **Greedy covering**: enumerate all `(w₁, w₂)` window pairs across `W¹ × W²`. Add pairs one at a time to the cover, each time picking the pair that covers the most yet-uncovered coarse matches. Stop when 90% of `M⁰_k` is covered.
4. **Per-window fine matching**: for each selected window pair, compute `D^{w₁}, D^{w₂} = MASt3R(I¹_{w₁}, I²_{w₂})`, then `M^{w₁,w₂}_k = fast_reciprocal_NN(D^{w₁}, D^{w₂})`.
5. **Principal-point preservation**: each window crop is transformed with a homography that keeps the original principal point at the centre of the cropped image, so the ViT sees a projection it was trained on.
6. **Reassembly**: map each window-pair's correspondences back to full-resolution coordinates; concatenate across pairs.

The set-cover reformulation is what makes this affordable. Naively, `|W¹| × |W²|` pairs is `O(n²)` in resolution; the greedy covering bounds it to `O(|M⁰_k|)` — in practice, a 4K×4K image pair needs only tens of window pairs to cover ≥90% of matches, not thousands.

The coarse pass is load-bearing: it tells the set-cover *where* the correspondences live, so the window-pair selection isn't brute-force.

## Why it wins

The paper provides two indirect pieces of evidence:

1. **DTU MVS zero-shot (Table 3e)** — MASt3R with coarse-to-fine achieves 0.403 accuracy, 0.344 completeness, 0.374 overall Chamfer vs [[dust3r]]'s 2.677/0.805/1.741. Cutting Chamfer by ~4.6× on DTU is attributable largely to being able to triangulate at full resolution rather than at the 512px bottleneck.
2. **AachenDayNight / InLoc (Table 4)** — MASt3R is tested at 1600×1200 input resolution (the standard Aachen query resolution). Without coarse-to-fine, a ViT-512 matcher cannot produce subpixel correspondences at that resolution. With it, MASt3R matches LightGlue/Kapture+R2D2's day performance (82.2/93.9/99.5 vs 91.3/97.0/99.5) — not best, but plausibly competitive. The specific Aachen "coarse-to-fine vs no-c2f" ablation is not reported.

No clean standalone ablation exists in the main paper — the authors treat coarse-to-fine as implementation detail. It's worth flagging that the evidence for this idea is weaker than for [[dust3r-matching-head_leroy2024]] or [[fast-reciprocal-nn-matching_leroy2024]].

## Preconditions & compatibility

- **Upstream**: base matcher with a bounded input resolution. Applies to any ViT-based matcher — [[mast3r|MASt3R]], [[dust3r|DUSt3R]], [[edstedt2025_roma-v2|RoMa v2]], LoFTR — all have token-count-dependent train resolutions.
- **Downstream consumer**: full-resolution correspondence set. Compatible with SfM, MVS, pose estimation, SLAM.
- **Independent of** [[metric-scale-pointmap-loss_leroy2024]] and [[dust3r-matching-head_leroy2024]] (orthogonal axes).
- **Uses** [[fast-reciprocal-nn-matching_leroy2024]] as the per-window matcher.
- **Not needed when** input resolution already fits the matcher's training resolution. Map-free (720×540 input) uses MASt3R at roughly its training scale and skips this step (Section 4.2).

## Trade-offs vs. the decomposed pipeline

Not applicable — `drop-in` scope. The coarse-to-fine wrapper replaces "single-shot at downsampled resolution" with "coarse plan + per-window fine match"; the downstream pipeline is unchanged.

## Open questions

- **Coverage threshold `90%`**: a hard-coded parameter. Does performance plateau at 80% with fewer windows? Untested in the paper.
- **Overlap `50%`**: fixed grid. For scenes with strongly non-uniform match density (e.g., sky vs cluttered floor), adaptive window placement might use fewer windows for the same coverage.
- **Interaction with matcher resolution training**: if the base matcher is trained on variable-resolution inputs (not just 512px), does the set-cover still help? Paper doesn't test MASt3R variants trained at other resolutions.
- **Principal-point preservation edge cases**: near the image edges the homography becomes degenerate (a 512px crop near the corner has principal point outside it). Paper doesn't specify the fallback.
- **Generalization to triplets / N-view**: the set-cover is pairwise. MVS triangulation would benefit from N-view windows, not just pairs. Open design question.
