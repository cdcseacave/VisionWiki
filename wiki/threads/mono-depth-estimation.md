---
title: Monocular Depth Estimation
type: thread
tags: [depth-estimation, monocular, metric-depth, relative-depth, foundation-model]
created: 2026-04-12
updated: 2026-04-12
sources: []
status: stub
---

## Working hypothesis
_(to be filled as papers are ingested)_

Monocular depth estimation has gone from a curiosity to a load-bearing
component in reconstruction pipelines — providing priors for sparse-view
NeRF/3DGS, initializing MVS, and enabling single-image 3D. This thread tracks:

1. The shift from relative/affine depth to metric depth (ZoeDepth, Metric3D,
   UniDepth, DepthAnythingV2).
2. Foundation-model approaches: large-scale pretraining on mixed datasets,
   zero-shot generalization (Depth Anything, MariGold, Depth Pro).
3. How mono depth integrates with multi-view methods: as a prior, a
   regularizer, or an initialization.
4. Accuracy ceilings: where does monocular depth reliably help, and where
   does it introduce systematic bias?
5. Video-consistent depth: temporal stability for dynamic scenes.

## Evidence
_(empty — will accumulate as papers are ingested)_

## Open questions
- Is metric monocular depth accurate enough to replace SfM depth maps for
  casual capture (phone-scale, ~10–50 images)?
- Do diffusion-based depth models (MariGold) offer real advantages over
  discriminative ones (DPT, Depth Anything) for downstream reconstruction?
- What's the right way to fuse mono depth priors into multi-view optimization
  without over-constraining the geometry?

## Related threads
- [[feed-forward-structure-from-motion]]
- [[radiance-field-evolution]]
