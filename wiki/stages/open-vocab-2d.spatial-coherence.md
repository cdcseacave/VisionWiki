---
title: Open-vocab-2d spatial coherence
type: stage
slug: open-vocab-2d.spatial-coherence
consumes: [image, frozen-backbone-features]
produces: [spatial-affinity-or-coherence-signal]
invariants: [patch-level-coherence-preserved]
provides_properties: [aggregation-friendly-spatial-signal]
requires_upstream_properties: [patch-token-features]
data_regime: [trident-style-composition]
tags: [dino, dinov3, self-similarity, sd-rpn]
created: 2026-04-18
updated: 2026-04-18
---

Provides a spatial affinity signal to propagate the semantic signal. DINO self-similarity is the classical filler; SD-RPN attention denoising is a candidate. Example fillers: [[dino-ibot-patch-ssl_oquab2023]], [[sd-rpn-attention-denoising_shi2026]].
