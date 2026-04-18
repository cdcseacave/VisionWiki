---
title: SfM geometric verification
type: stage
slug: sfm.geometric-verification
consumes: [feature-matches]
produces: [verified-matches-and-pairwise-geometric-relations]
invariants: [degeneracies-caught]
provides_properties: [robust-to-panorama-planar-configs]
requires_upstream_properties: [feature-matches-raw]
data_regime: [any-sfm]
tags: [sfm, ransac, multi-model, colmap]
created: 2026-04-18
updated: 2026-04-18
---

Filters and geometrically verifies feature matches. Multi-model RANSAC (homography + epipolar) is the COLMAP-established filler; single-model F-matrix RANSAC was the predecessor. Example fillers: [[colmap-incremental-sfm_schonberger2016]].
