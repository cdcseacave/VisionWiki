---
title: Depth Anything v3 (DAv3)
type: method
tags: [monocular-depth, feed-forward-3d, foundation-model, 3dgs-prediction, per-pixel-gaussian-head, nvidia]
created: 2026-04-21
updated: 2026-04-21
sources: [wiki/papers/shen2026_lyra2.md]
status: stub
---

## TL;DR

**Stub.** Depth Anything v3 (DAv3) is a feed-forward 3D foundation model that, in addition to the monocular-depth capability of its predecessors, predicts **per-pixel 3D Gaussian Splatting attributes** from RGB input — turning depth estimation into feed-forward 3DGS reconstruction. Used by [[shen2026_lyra2|Lyra 2.0]] as:

1. **The depth backbone** for per-frame geometry in its 3D cache (feeding `[[per-frame-3d-cache-retrieval_shen2026]]`).
2. **The feed-forward 3DGS reconstructor** that lifts generated videos to 3D — with a Lyra-specific modification via `[[downsampled-gaussian-dpt-head_shen2026]]` (k=2 strided head for 4× Gaussian reduction).
3. **Fine-tuned on generated-scene data** (3,000 autoregressive one-minute videos from DL3DV) to close the domain gap between real-capture training and generative-output deployment. The fine-tuning recipe inherits from [[bahmani2025_lyra|Lyra 1]].

## Why this stub exists

DAv3 is cited as `[58]` in [[shen2026_lyra2|Lyra 2.0]] and is load-bearing at two distinct stages of the pipeline. When a dedicated DAv3 paper ingest lands, this stub should grow to include:

- Architecture (likely extends DAv1/v2's DPT decoder with a Gaussian head).
- Training regime (scale of labeled + unlabeled data; self-supervised bootstrap).
- Performance numbers vs DAv2 on mono-depth benchmarks and vs pixelSplat / MVSplat / NoPoSplat on feed-forward 3DGS benchmarks.

## Relation to Depth Anything v1/v2

- [[depthanything|Depth Anything v1/v2]] — relative depth foundation model (Yang et al. 2024) on ~62M unlabeled images.
- DAv3 extends this lineage with the **per-pixel Gaussian prediction head** — the reason it appears in a radiance-fields workflow, not just in `[[mono-depth-estimation]]`.

## References

- [[shen2026_lyra2]] §4.4 + App. A.4 — modifies DAv3's Gaussian DPT head + fine-tunes on generated data.
- [[bahmani2025_lyra|Lyra 1]] — precedent for the fine-tune-on-generated-data recipe.

> [!needs-source] DAv3 primary paper not yet ingested.
