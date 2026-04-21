---
title: Reciprocal nearest-neighbor matching (feature-map pair)
type: stage
slug: feature-matching.reciprocal-matching
consumes: [dense-feature-map-pair, unit-norm-descriptors]
produces: [reciprocal-correspondence-set]
invariants: [symmetric-under-image-swap, cycle-consistency]
provides_properties: [cycle-consistency, outlier-filtered-matches]
requires_upstream_properties: [pixel-aligned-descriptor-map, shared-metric-space]
data_regime: [matcher-output-bounded-resolution]
tags: [reciprocal, nn-matching, dense-matcher, cycle-consistency]
created: 2026-04-21
updated: 2026-04-21
---

Converts a pair of dense per-pixel descriptor maps into a set of mutual (reciprocal) correspondences. A correspondence `(i, j)` belongs to the output iff pixel `i` in image 1's nearest neighbor in image 2 is `j` **and** `j`'s nearest neighbor back in image 1 is `i` — the cycle `i → j → i` closes. The naive realization is `O(W²H²)` (all-pairs distances); fast realizations exploit sub-sampling + iteration to trade recall for speed and for implicit outlier filtering.

Sits downstream of any dense matcher or dense descriptor head ([[feature-matching.task-head]]) and upstream of geometry/pose stages that consume a correspondence set. Example fillers: [[fast-reciprocal-nn-matching_leroy2024]].
