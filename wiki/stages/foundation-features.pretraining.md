---
title: Foundation-features pretraining
type: stage
slug: foundation-features.pretraining
consumes: [curated-training-corpus]
produces: [frozen-vit-backbone-with-dense-features]
invariants: [features-spatially-coherent-and-semantically-meaningful]
provides_properties: [downstream-task-heads-train-in-hours]
requires_upstream_properties: []
data_regime: [ssl-pretraining]
tags: [dinov2, dinov3, ssl, ibot, gram-anchoring]
created: 2026-04-18
updated: 2026-04-18
---

Self-supervised pretraining of the ViT backbone. DINO+iBOT (DINOv2) + Gram anchoring (DINOv3) is the 2023–2025 recipe. Example fillers: [[dino-ibot-patch-ssl_oquab2023]], [[dinov3-gram-anchoring_simeoni2025]].
