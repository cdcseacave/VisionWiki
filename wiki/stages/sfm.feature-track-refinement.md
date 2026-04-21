---
title: SfM feature-track refinement
type: stage
slug: sfm.feature-track-refinement
consumes: [coarse-feature-tracks, per-view-image-patches]
produces: [refined-subpixel-feature-tracks, per-view-track-uncertainty]
invariants: [track-identity-preserved, subpixel-accuracy-improved]
provides_properties: [geometric-consistency-across-views, per-keypoint-uncertainty]
requires_upstream_properties: [coarse-tracks-from-detector-free-or-detector-based-matching]
data_regime: [any-sfm]
tags: [sfm, feature-track, pixsfm, detector-free-sfm, multi-view-transformer]
created: 2026-04-21
updated: 2026-04-21
---

Refines 2D keypoint locations within a multi-view feature track after initial mapping has produced coarse poses. Distinct from [[sfm.iterative-triangulation-ba]], which edits 3D points + poses (not 2D locations), and from [[feature-matching.task-head]], which produces raw pairwise matches (not multi-view tracks).

Example fillers:

- [[multi-view-transformer-track-refinement_he2023]] — transformer-based attention over `p×p` patches from all views, reference-location search over a `w×w` grid, minimum-variance track selection.
- *PixSfM (Lindenberger 2021)* — feature-metric BA with CNN features; cost is O(#observations) patches in memory. Heaviest classical filler.
- *PatchFlow (Dusmanu 2020)* — dense flow within local patches + multi-view 2D refinement; pairwise flow cost.

Valid-filler constraints: filler must preserve the track graph topology (no edit to which 2D keypoint belongs to which track — that is the job of [[sfm.track-topology-adjustment]]). Output uncertainty is optional but enables inverse-variance weighting downstream.
