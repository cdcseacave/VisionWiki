---
title: Radiance-fields partitioning
type: stage
slug: radiance-fields.partitioning
consumes: [camera-positions, scene-bounds, sparse-points]
produces: [per-cell-camera-set, per-cell-seed-points, cell-geometry]
invariants: [cells-tile-scene-with-controlled-overlap]
provides_properties: [per-cell-parallel-training-possible, seamless-merge-at-boundaries]
requires_upstream_properties: [manhattan-or-ground-plane-alignment]
data_regime: [unbounded-scene, city-scale, outdoor-default]
tags: [3dgs, city-scale, partitioning]
created: 2026-04-18
updated: 2026-04-18
---

Splits a large scene into trainable cells. Airspace-aware visibility criterion + progressive 4-step partition (VastGaussian) is the canonical filler. Fillers must produce a post-training discard rule so over-expanded cells don't duplicate Gaussians. Example fillers: [[airspace-aware-visibility-partitioning_lin2024]], [[progressive-4-step-partition_lin2024]].
