---
title: Radiance-fields loss balancing
type: stage
slug: radiance-fields.loss-balancing
consumes: [3d-primitives, photometric-loss-term, geometric-loss-term]
produces: [weighted-total-loss, per-primitive-weight-field]
invariants: [gradient-remains-well-conditioned, weights-stay-bounded]
provides_properties: [per-primitive-loss-attribution]
requires_upstream_properties: [separable-photometric-geometric-losses]
data_regime: [static-scene, posed-input, bounded-or-unbounded]
tags: [3dgs, loss-balancing, confidence]
created: 2026-04-18
updated: 2026-04-18
---

## Description

Balance photometric and geometric losses at primitive or pixel granularity. In the simplest case a hand-tuned global weight ratio; at the current frontier a per-primitive learned confidence (CoMe). The stage assumes downstream integration (BA / TSDF fusion / mesh extraction) can consume a weighted loss; degraded to a constant when the downstream is agnostic.

## Example fillers

- [[per-gaussian-self-supervised-confidence_radl2026]] — learnable per-primitive scalar.
- Hand-tuned global weight ratio — baseline pre-2026.

## Valid-filler notes

Fillers must produce a per-primitive or per-pixel weight field; global-constant fillers are legal but trivial. A filler that fuses photometric and geometric losses at construction time (not at balancing time) is a **stage-collapse** and belongs in a different stage — this stage assumes the losses are still separable on arrival.
