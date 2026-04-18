---
title: Active-reconstruction next-best-view scoring
type: stage
slug: active-reconstruction.next-best-view-scoring
consumes: [candidate-viewpoints, uncertainty-field]
produces: [ranked-viewpoints-by-expected-information-gain]
invariants: [realtime-closed-form-or-cheap]
provides_properties: [planner-compatible-output]
requires_upstream_properties: [uncertainty-representation-available]
data_regime: [active-capture]
tags: [gauss-mi, nbv, shannon-mi]
created: 2026-04-18
updated: 2026-04-18
---

Scores novel viewpoints by expected information gain. Closed-form Shannon MI (GauSS-MI) is the real-time filler. Example fillers: [[closed-form-mi-next-best-view-3dgs_xie2025]].
