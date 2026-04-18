---
title: Radiance-fields compression
type: stage
slug: radiance-fields.compression
consumes: [trained-primitive-set]
produces: [compressed-primitives-with-codebook-or-quantized-attributes]
invariants: [render-quality-preserved-within-dB-budget]
provides_properties: [deployment-size-reduced, codebook-lookup-at-render-time]
requires_upstream_properties: [training-converged]
data_regime: [post-training-storage]
tags: [3dgs, compression, vq, codebook]
created: 2026-04-18
updated: 2026-04-18
---

Post-training (or QAT) compression of primitive attributes. Codebook VQ on non-position attributes (EA-3DGS), pruning by contribution score, or joint quantization are fillers. Compatible with partitioning (stacks per submap). Example fillers: [[codebook-vq-compression_guo2025]].
