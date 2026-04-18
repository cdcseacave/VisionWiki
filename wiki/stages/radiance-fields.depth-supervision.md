---
title: Radiance-fields depth supervision
type: stage
slug: radiance-fields.depth-supervision
consumes: [rendered-depth, external-depth-prior]
produces: [depth-supervision-loss]
invariants: [uncertainty-weighting-available]
provides_properties: [metric-scale-preservation, bias-handling]
requires_upstream_properties: [mvs-or-mono-depth-source]
data_regime: [static-scene, posed-input]
tags: [mvs, depth-supervision, kim2025, confidence-weighted]
created: 2026-04-18
updated: 2026-04-18
---

Applies external depth priors (MVS, monocular) to 3DGS training. Must handle α-composited depth bias; median-based fillers (Kim 2025) do this correctly. Natural composition with loss-balancing ([[radiance-fields.loss-balancing]]). Example fillers: [[median-depth-relative-loss_kim2025]].
