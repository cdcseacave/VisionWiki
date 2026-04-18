---
title: Lifting — per-primitive identity
type: stage
slug: lifting-foundation-models.per-primitive-identity
consumes: [primitives, per-view-sam-masks]
produces: [per-primitive-identity-vector]
invariants: [cross-view-id-consistency-within-instance]
provides_properties: [instance-editable-scene]
requires_upstream_properties: [sam-or-sam3-masks-available]
data_regime: [per-scene, static-or-in-the-wild]
tags: [gaussian-grouping, identity-encoding, sam-lifting]
created: 2026-04-18
updated: 2026-04-18
---

Learns per-primitive identity vectors supervised by 2D masks. Gaussian Grouping's canonical filler; Seg-Wild's multi-dim-embedding variant refines it. Example fillers: [[gaussian-grouping-identity-encoding_ye2024]], [[multi-dimensional-per-gaussian-embeddings_bao2025]].
