---
title: Relighting inverse rendering
type: stage
slug: relighting.inverse-rendering
consumes: [images, primitive-set, environment-map]
produces: [disentangled-albedo-normal-specular]
invariants: [physics-based-rendering-equation-respected]
provides_properties: [relightable-output]
requires_upstream_properties: [primitives-carry-sdf-or-surface-info]
data_regime: [reflective-objects, glossy-scenes]
tags: [relighting, inverse-rendering, asg, sdf-gaussians]
created: 2026-04-18
updated: 2026-04-18
---

Disentangles material properties from lighting by following the rendering equation explicitly. ASG decomposition (AniSDF) or per-Gaussian SDF+SH (Zhu 2025) are fillers. Example fillers: [[per-gaussian-discretized-sdf_zhu2025]], [[fused-granularity-dual-hash-grid-plus-asg_gao2025]].
