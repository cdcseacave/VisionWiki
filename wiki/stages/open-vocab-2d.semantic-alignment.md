---
title: Open-vocab-2d semantic alignment
type: stage
slug: open-vocab-2d.semantic-alignment
consumes: [image, text-prompt]
produces: [per-region-similarity-with-text]
invariants: [zero-shot-across-vocabulary]
provides_properties: [text-region-correspondence]
requires_upstream_properties: [frozen-vision-and-text-encoder]
data_regime: [any-image]
tags: [clip, siglip, contrastive]
created: 2026-04-18
updated: 2026-04-18
---

Text-image alignment via contrastive dual encoders. CLIP is the canonical filler; SigLIP / SAM 3 (native concept prompts) are candidates. Example fillers: [[contrastive-dual-encoder_radford2021]].
