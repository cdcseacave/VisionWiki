---
title: Feed-Forward Structure from Motion
type: thread
tags: [sfm, pose-estimation, feed-forward, dust3r, mast3r, rommav2, transformer]
created: 2026-04-12
updated: 2026-04-12
sources: []
status: stub
---

## Working hypothesis
_(to be filled as papers are ingested)_

Classical SfM (COLMAP-style) is iterative: detect features, match, estimate
geometry, bundle-adjust. A new wave of methods replaces part or all of this
pipeline with feed-forward neural networks that predict 3D structure and/or
camera poses in a single pass. This thread tracks:

1. The spectrum from "replace one SfM stage" (learned matching) to "replace
   the whole pipeline" (DUSt3R, MASt3R, Fast3R).
2. How feed-forward pose estimation compares to optimization-based BA in
   accuracy, robustness, and speed.
3. Scaling behavior: do these methods generalize across scenes, or do they
   need per-domain training?
4. Integration with downstream tasks: can feed-forward poses initialize
   Gaussian splatting or NeRF without COLMAP?
5. The role of foundation models and large-scale pretraining in making
   feed-forward SfM viable.

## Evidence
_(empty — will accumulate as papers are ingested)_

## Open questions
- Can feed-forward methods match COLMAP accuracy on large-scale outdoor scenes?
- Is the endgame a single model that goes from images → posed point cloud,
  or will hybrid pipelines (learned init + classical refinement) dominate?
- How sensitive are these methods to domain shift (indoor → outdoor, object → scene)?
- What's the minimum viable training data for a new domain?

## Related threads
- [[radiance-field-evolution]]
- [[mono-depth-estimation]]
