---
title: SfM track topology adjustment
type: stage
slug: sfm.track-topology-adjustment
consumes: [current-track-graph, refined-poses, refined-3d-points, unregistered-2d-keypoints]
produces: [edited-track-graph, extended-tracks]
invariants: [added-observations-pass-reprojection-threshold, no-cycle-introduction]
provides_properties: [track-completeness-recovered, outliers-rejected]
requires_upstream_properties: [post-BA-reconstruction-available]
data_regime: [final-polish-of-incremental-or-coarse-to-fine-sfm]
tags: [sfm, track-editing, track-merging, detector-free-sfm]
created: 2026-04-21
updated: 2026-04-21
---

Edits the **structure of the track graph** (which 2D observations belong to which 3D track) after a BA pass. Complements [[sfm.iterative-triangulation-ba]], which edits parameters (poses + points) but keeps the track graph fixed. Useful when the initial registration threshold was too strict and missed observations that now, with refined poses, clearly belong to existing 3D points.

Operations performed:

- **Complete** broken tracks: for a 3D point `P_j`, find unregistered 2D keypoints in views where the reprojection now falls below a looser threshold (e.g., `3·ε`) and add them to `T_j`.
- **Merge** tracks: two tracks whose 3D points lie within a small reprojection + 3D distance tolerance are unified.
- **Filter** outliers: observations whose reprojection exceeds `ε` after BA are dropped.

Example fillers:

- [[iterative-ba-plus-track-topology-adjustment_he2023]] — BA ↔ TA alternation `T=5` iterations; reprojection-based merge + completion + outlier filter.

Valid-filler constraints: must be callable as a post-BA step (needs converged-ish poses); must not invalidate track IDs that downstream consumers already depend on (use append-only merges).
