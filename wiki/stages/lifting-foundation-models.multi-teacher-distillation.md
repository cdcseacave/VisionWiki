---
title: Lifting — multi-teacher distillation
type: stage
slug: lifting-foundation-models.multi-teacher-distillation
consumes: [primitives, language-fm-teacher, geometry-fm-teacher]
produces: [co-trained-primitive-fields]
invariants: [teacher-balance-controllable]
provides_properties: [geometry-regularizes-semantics-and-vice-versa]
requires_upstream_properties: [multiple-foundation-model-teachers]
data_regime: [one-stage-training]
tags: [langsvr, dual-distillation, voxel]
created: 2026-04-18
updated: 2026-04-18
---

Jointly distills multiple foundation model teachers (CLIP + mono-depth in LangSVR) into a shared primitive representation. Example fillers: [[langsvr-four-field-dual-distillation_wu2026]].
