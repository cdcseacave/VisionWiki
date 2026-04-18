---
title: Radiance-fields sparse-view regularization
type: stage
slug: radiance-fields.sparse-view-regularization
consumes: [sparse-views, voxel-grid-or-mvs-points]
produces: [regularization-loss-for-sparse-view]
invariants: [over-fit-prevention]
provides_properties: [works-at-3-to-10-views]
requires_upstream_properties: [external-geometry-prior-available]
data_regime: [sparse-view-3-to-10-views]
tags: [3dgs, sparse-view, dust3r-init, voxel-guided]
created: 2026-04-18
updated: 2026-04-18
---

Extra regularization needed when training on few views. Voxel grids from DUSt3R-MVS (DroneSplat) provide structural constraints that keep Gaussians near plausible surfaces. Example fillers: [[mvs-voxel-guided-sparse-view-regularization_tang2025]].
