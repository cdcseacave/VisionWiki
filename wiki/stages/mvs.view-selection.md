---
title: MVS view selection
type: stage
slug: mvs.view-selection
consumes: [image-set, reference-view, photometric-geometric-costs]
produces: [per-pixel-or-per-ref-source-view-subset]
invariants: [adaptation-to-scene-coverage]
provides_properties: [heterogeneous-overlap-robustness]
requires_upstream_properties: [initial-cost-volume-or-matches]
data_regime: [internet-photo-collections, classical-mvs]
tags: [mvs, patchmatch, colmap, view-selection]
created: 2026-04-18
updated: 2026-04-18
---

Per-pixel source-view selection within PatchMatch MVS. Probabilistic graphical model (COLMAP) is the load-bearing filler. Example fillers: [[per-pixel-probabilistic-view-selection_schonberger2016]].
