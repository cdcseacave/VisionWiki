---
title: SfM multi-camera rig extrinsic refinement
type: stage
slug: sfm.rig-extrinsic-refinement
consumes: [per-camera-poses, rig-structure-definition]
produces: [refined-camera-to-rig-transforms, refined-rig-trajectory]
invariants: [rig-rigidity-preserved]
provides_properties: [joint-pose-plus-calibration]
requires_upstream_properties: [rig-structure-known-or-initial-guess]
data_regime: [autonomous-driving, multi-camera-rig]
tags: [sfm, multi-camera-rig, cusfm]
created: 2026-04-18
updated: 2026-04-18
---

Refines camera-to-vehicle (or camera-to-rig) extrinsics jointly with the rig trajectory. Rigid-rig assumption; time-varying calibration unsupported. Example fillers: [[cusfm-slam-prior-factor-graph_yu2025]].
