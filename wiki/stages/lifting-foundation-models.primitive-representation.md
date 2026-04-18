---
title: Lifting — primitive representation (composite stage for LangSVR-style pipelines)
type: stage
slug: lifting-foundation-models.primitive-representation
consumes: [posed-images]
produces: [appearance-density-feature-confidence-fields-per-primitive]
invariants: [fields-co-trained-jointly]
provides_properties: [one-stage-pipeline]
requires_upstream_properties: [svraster-or-equivalent-voxel-base]
data_regime: [one-stage-training]
tags: [langsvr, four-fields, voxel]
created: 2026-04-18
updated: 2026-04-18
---

Composite stage where appearance + density + feature + confidence fields share a primitive and are co-trained. Example fillers: [[langsvr-four-field-dual-distillation_wu2026]].
