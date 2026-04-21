---
title: Coarse-to-fine detector-free SfM bridge (match quantization + coarse mapping)
type: idea
source_paper: wiki/papers/he2023_detector-free-sfm.md
also_in: []

scope: bridge
stages: [feature-matching.task-head, sfm.view-graph-construction]
collapses: []
splits_into: []
rewrites: {}

inputs: [pairwise-detector-free-matches, coarse-matcher-stride-r]
outputs: [multi-view-consistent-sparse-keypoints, coarse-sfm-model]
assumptions: [detector-free-matcher-available, coarse-stride-known, static-scene]
requires_upstream_property: [matcher-coarse-matches-at-common-stride-across-pairs]
requires_downstream_property: [colmap-compatible-keypoint-format, incremental-mapping-accepts-external-keypoints]
learned_params: []
failure_modes: [r-too-coarse-kills-fine-detail, r-too-small-fails-to-merge-pair-dependent-matches, matcher-with-per-pair-adaptive-stride-breaks-quantization]

requires: []
unlocks: [multi-view-transformer-track-refinement_he2023, iterative-ba-plus-track-topology-adjustment_he2023]
co_requires: [multi-view-transformer-track-refinement_he2023, iterative-ba-plus-track-topology-adjustment_he2023]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, detector-free, loftr, match-quantization, coarse-to-fine, bridge]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
A detector-free matcher (LoFTR-class) runs on image pairs and emits sub-pixel correspondences at its native coarse stride `r` (r=8 for LoFTR). On a triple `(I_i, I_j, I_k)`, matching `I_i↔I_j` vs `I_j↔I_k` places the *same* 3D point at *different* sub-pixel locations in `I_j`, so feature tracks fragment and classical incremental SfM cannot form them. The quantization bridge applies the grid-rounding operator `⌊x/r⌋·r` to every matched location before feeding it to the mapper. All sub-pixel matches whose original locations lie within one `r×r` cell collapse to one shared quantized keypoint, re-establishing cross-view identity. The resulting quantized keypoints are fed as if they were detected (same file format, same track graph construction rules) into a standard COLMAP incremental mapper — producing a coarse SfM model whose pose accuracy is bounded by `r` but whose track topology is correct.

Nothing is learned in the bridge itself; the matcher is learned (LoFTR/AspanFormer/MatchFormer), the mapper is classical. The bridge is a **type conversion**: dense per-pair sub-pixel flow → sparse multi-view consistent keypoint set.

## Why it wins
Causal story: detector-free matchers generalize better on low-texture, but their per-pair sub-pixel output is multi-view-inconsistent (Fig. 2 of [[he2023_detector-free-sfm]]). Merge strategies (earlier work by [46]) lengthen tracks but smear accuracy. Quantization at the matcher's native coarse stride is the minimum-information merge — it discards exactly the sub-pixel residual that was pair-dependent and keeps everything the matcher actually resolved across views. Refinement (via co-required ideas) then recovers the sub-pixel accuracy on top.

Specific ablation: Table 3 (2) of [[he2023_detector-free-sfm]] shows the coarse SfM alone (no refinement) achieves 55.92% AUC@2cm on ETH3D — already above the 42.12% best detector-based baseline without refinement on the same metric. The full pipeline reaches 80.38% but the bridge alone delivers most of the robustness gain. Table 3 (1) isolates the quantization ratio: `r=8` outperforms `r=4` (81.58 vs 80.38 @ 1cm) but `r=4` triples runtime (718s vs 557s) — `r=8` is the operating point.

## Preconditions & compatibility
- **Upstream**: matcher must expose matches at a *pair-independent* coarse grid. LoFTR, AspanTransformer, MatchFormer all do (1/8-stride feature maps). Adaptive-stride matchers would break the bridge.
- **Downstream**: mapper must accept externally-supplied keypoints and track initialization. COLMAP does; GLOMAP does; InstantSfM does (it's COLMAP-on-GPU).
- **Bundle**: the bridge alone yields only coarse poses; without [[multi-view-transformer-track-refinement_he2023]] and [[iterative-ba-plus-track-topology-adjustment_he2023]], the pose error floor is 5° / 13mm (Fig. 3). All three travel together.

## Pipeline-shape implications
Bridge nodes are marked `(bridge)` in thread SOTA pipelines (§2.1). This bridge inserts a single node between [[feature-matching.task-head]] output and [[sfm.view-graph-construction]] input; the rest of the pipeline's DAG is unchanged. Any thread adopting it must honor the bundle — the other two nodes of the DetectorFreeSfM sub-DAG (refinement + topology adjustment) are required downstream.

## Trade-offs vs. the decomposed pipeline
Not applicable — `bridge` scope. The bridge is additive: it doesn't remove a classical stage, it retypes an existing boundary so detector-free matchers can feed a classical mapper.

## Open questions
- Would **foundation-features-based** matchers (RoMa v2 on DINOv3, [[edstedt2025_roma-v2|RoMa v2]]) work as a drop-in matcher replacement? The paper tests LoFTR / AspanFormer / MatchFormer but not RoMa — a concrete synthesis bet in [[foundation-features-for-geometry]] / [[feed-forward-structure-from-motion]].
- Does the choice of `r` interact with scene scale? The paper tunes once on ETH3D; outdoor IMC uses the same `r=8`. Untested on very-wide-baseline outdoor.
- Could a **learned** quantization (e.g., attention-based merge that preserves sub-pixel accuracy in high-texture regions and quantizes aggressively in low-texture) outperform fixed `r`? Not proposed in the paper.
