---
title: Active-reconstruction uncertainty representation
type: stage
slug: active-reconstruction.uncertainty-representation
consumes: [3dgs-in-progress]
produces: [per-primitive-uncertainty-fields]
invariants: [uncertainty-differentiable-through-render]
provides_properties: [mi-or-entropy-computable]
requires_upstream_properties: [3dgs-expressing-parameter-variance]
data_regime: [active-capture]
tags: [gauss-mi, color-variance, information-theory]
created: 2026-04-18
updated: 2026-04-18
---

Represents per-primitive uncertainty so information gain can be computed. GauSS-MI's learned color variance is the canonical filler. Example fillers: [[closed-form-mi-next-best-view-3dgs_xie2025]].
