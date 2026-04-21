---
title: Iterative BA + track topology adjustment (DetectorFreeSfM)
type: idea
source_paper: wiki/papers/he2023_detector-free-sfm.md
also_in: []

scope: stage-split
stages: [sfm.iterative-triangulation-ba]
collapses: []
splits_into: [sfm.iterative-triangulation-ba, sfm.track-topology-adjustment]
rewrites: {}

inputs: [refined-tracks, current-poses, current-3d-points, unregistered-2d-keypoints]
outputs: [refined-poses, refined-3d-points, extended-track-graph]
assumptions: [refined-2d-tracks-available, post-BA-pose-accuracy-sufficient-for-reprojection-gating, static-scene]
requires_upstream_property: [coarse-sfm-with-refined-tracks]
requires_downstream_property: [downstream-consumes-track-graph-or-3d-points-only]
learned_params: []
failure_modes: [topology-growth-stalls-after-2-iterations, looser-merge-threshold-dataset-dependent, symmetric-scenes-merge-incorrectly]

requires: [multi-view-transformer-track-refinement_he2023]
unlocks: []
co_requires: [coarse-to-fine-detector-free-sfm-bridge_he2023, multi-view-transformer-track-refinement_he2023]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, track-topology, ba, triangulation, detector-free-sfm]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism
Classical [[sfm.iterative-triangulation-ba]] alternates triangulation and BA: re-triangulate tracks with current poses, then re-optimize poses + points on reprojection. Both steps edit **parameters only** (poses, 3D point coordinates); the track graph (which 2D observation belongs to which 3D track) is frozen at what incremental mapping produced. When the initial registration thresholds were too strict, observations that *should* belong to existing tracks stay unregistered forever.

This idea adds a second step that edits **the track graph itself**, giving a stage split:

1. *BA sub-step*: plain geometric reprojection BA on refined tracks (reprojection-only, no feature-metric) using Ceres with the Triggs et al. 1999 formulation. Small memory (no patches stored).
2. *Track-topology adjustment (TA)*: project every refined 3D point `P_j` back to every image using current poses; for each image, find unregistered 2D keypoints whose reprojection distance to `P_j` is below a looser threshold `ε_merge > ε` — add them to `T_j` (track completion). Merge tracks whose projections coincide. Filter outliers above `ε_BA` after BA. Re-triangulate completed tracks.

Alternate BA ↔ TA five times in the reported configuration. Outlier filter runs after each pass (following [47, 63, 44] — COLMAP's filter convention).

## Why it wins
Causal story: detector-free matches produce many candidate observations per 3D point, but the conservative registration threshold used during coarse incremental mapping (to prevent outlier contamination) rejects many that *would* pass once poses are refined. The classical triangulation-BA loop cannot recover them because it doesn't re-visit registration decisions. TA explicitly re-registers them after each BA pass, unlocking track completeness that pure parameter refinement cannot reach.

Isolating ablation (Table 3 (4) of [[he2023_detector-free-sfm]], ETH3D triangulation):
- Full: 80.38 / 89.01 / 95.83 @ 1cm/2cm/5cm, 3.73 / 11.07 / 29.54 completeness.
- w/o topology adjustment: 75.78 / 85.47 / 94.07, 4.07 / 12.18 / 32.77. Accuracy drops 4.6 points on the strict 1cm threshold; completeness actually rises slightly (more unfiltered observations) — the point is that TA improves the accuracy-completeness trade-off on hard thresholds.

Iterations ablation (Table 3 (2)): 1 iter → 77.62 @ 1cm; 2 iter → 80.38; 3 iter → 81.26. Diminishing returns after 2, so the paper uses 2 as operating point (cost-weighted). Classical iterative-triangulation-BA without TA would stall earlier because parameter refinement alone saturates the information content of the frozen track graph.

## Preconditions & compatibility
- **Upstream**: needs refined tracks from [[multi-view-transformer-track-refinement_he2023]]. Without refined sub-pixel locations the TA reprojection gating rejects most completion candidates.
- **Downstream**: refined reconstruction is consumed the same way as any COLMAP output (poses + sparse point cloud); downstream doesn't need to be aware of the topology edits.
- **Bundle**: co-required with A1 + A2. The DetectorFreeSfM pipeline is one unit; partial adoption breaks the accuracy floor.
- **Thresholds**: `ε = 3px` reprojection for BA outlier filter; `ε_merge` ≈ 2ε (soft merge threshold); looser `ε_complete` for completion. All reported values are tuned on ETH3D; cross-dataset robustness is claimed but not fully ablated.

## Pipeline-shape implications
Stage split: adopting threads must represent the two-node structure `sfm.iterative-triangulation-ba` → `sfm.track-topology-adjustment`, with TA reading the output of the current BA pass and feeding its edited track graph back into the next BA pass. A single-node [[sfm.iterative-triangulation-ba]] filler cannot represent this; the split is load-bearing for the 4.6% accuracy gain.

## Trade-offs vs. the decomposed pipeline
The split formalizes what COLMAP already does somewhat informally (its outlier filter is a weak form of topology editing). The decomposed form makes TA ablatable and separately tunable — which is precisely how the paper isolates its 4.6% contribution. The cost is one more stage page in the taxonomy; the benefit is that future filler ideas for track topology (e.g., learned track merging, global track-graph optimization) have a legitimate slot to live in.

## Open questions
- **Cross-dataset threshold robustness**: the paper uses fixed `ε` across ETH3D / IMC / Texture-Poor — does it break on scenes with very different pose scales (aerial, underwater)?
- **Interaction with [[mono-depth-normal-constrained-incremental-sfm_pataki2025]]**: MP-SfM's forward-backward depth consistency check is a different post-BA filter on registration; can TA and that check compose, or do they conflict?
- **Global TA**: currently TA is a local reprojection gate per 3D point. Would a global track-graph optimization (joint merge + split + complete minimizing a global objective) do better? Open.
