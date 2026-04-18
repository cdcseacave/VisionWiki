---
title: Lifting — view reliability gating
type: stage
slug: lifting-foundation-models.view-reliability-gating
consumes: [multi-view-observations, per-primitive-confidence]
produces: [gated-view-contribution]
invariants: [low-reliability-views-downweighted]
provides_properties: [transient-occluder-robustness]
requires_upstream_properties: [confidence-field-trained-jointly]
data_regime: [in-the-wild, occlusion-heavy]
tags: [langsvr, confidence, gating]
created: 2026-04-18
updated: 2026-04-18
---

Per-primitive confidence gates which views contribute to the loss. Example fillers: [[langsvr-four-field-dual-distillation_wu2026]].
