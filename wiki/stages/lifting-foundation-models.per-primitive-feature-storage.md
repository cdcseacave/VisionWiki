---
title: Lifting — per-primitive feature storage
type: stage
slug: lifting-foundation-models.per-primitive-feature-storage
consumes: [high-dim-foundation-features, primitives]
produces: [compressed-per-primitive-latent, query-time-decoder]
invariants: [memory-bounded-at-1m-plus-primitives]
provides_properties: [cosine-query-at-inference]
requires_upstream_properties: [foundation-features-per-view]
data_regime: [per-scene-3dgs]
tags: [langsplat, autoencoder, clip-distillation, per-scene]
created: 2026-04-18
updated: 2026-04-18
---

Compresses high-dim features (512-D CLIP) into per-primitive latents via a scene-specific autoencoder. LangSplat's canonical filler. Example fillers: [[langsplat-per-scene-autoencoder_qin2024]].
