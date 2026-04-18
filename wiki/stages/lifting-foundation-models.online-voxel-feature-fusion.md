---
title: Lifting — online voxel feature fusion
type: stage
slug: lifting-foundation-models.online-voxel-feature-fusion
consumes: [rgbd-stream, fm-feature-per-frame]
produces: [per-voxel-running-feature-average]
invariants: [one-pass-per-frame-online]
provides_properties: [multimodal-shared-embedding-queries]
requires_upstream_properties: [tsdf-backend-compatible]
data_regime: [online-robotics]
tags: [conceptfusion, tsdf-fusion, multimodal]
created: 2026-04-18
updated: 2026-04-18
---

Voxel-hashed TSDF extended with per-voxel running feature average. ConceptFusion's canonical filler. Example fillers: [[voxel-tsdf-multimodal-fusion_jatavallabhula2023]].
