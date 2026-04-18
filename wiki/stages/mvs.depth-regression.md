---
title: MVS depth regression
type: stage
slug: mvs.depth-regression
consumes: [regularized-cost-volume]
produces: [subpixel-depth-map]
invariants: [smoothness-from-3d-cnn]
provides_properties: [expected-depth-via-soft-argmin]
requires_upstream_properties: [cost-volume-regularized]
data_regime: [learned-mvs]
tags: [mvs, mvsnet, soft-argmin, 3d-cnn]
created: 2026-04-18
updated: 2026-04-18
---

Converts cost volume to depth via 3D-CNN regularization + soft-argmin. Example fillers: [[plane-sweep-variance-cost-volume_yao2018]].
