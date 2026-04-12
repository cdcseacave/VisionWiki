---
title: Radiance Field Evolution
type: thread
tags: [nerf, 3dgs, differentiable-rendering, novel-view-synthesis]
created: 2026-04-11
updated: 2026-04-12
sources: []
status: stub
---

## Working hypothesis
_(to be filled as papers are ingested)_

The line from NeRF (2020) through to 3D Gaussian Splatting (2023) and beyond
is the dominant story in neural scene representation for photogrammetry.
This thread tracks:

1. What each generation actually optimizes (MLP weights vs explicit primitives).
2. Tradeoffs in training time, inference speed, memory, and reconstruction fidelity.
3. How well each integrates with classical photogrammetry pipelines (SfM priors,
   camera calibration, sparse/dense correspondence).
4. Where the field is headed next (dynamic scenes, generative priors, mesh extraction).

## Evidence
_(empty — will accumulate as papers are ingested)_

## Open questions
- Which representation wins for *editable* geometry vs pure novel-view synthesis?
- How does 3DGS composition interact with traditional bundle adjustment?
- What's the current best path from a Gaussian splat to a usable textured mesh?

## Related threads
- [[gaussian-to-mesh-pipelines]] — the downstream question of what to do with Gaussians
- [[feed-forward-structure-from-motion]] — feed-forward methods that may replace the SfM stage radiance fields depend on
- [[mono-depth-estimation]] — mono depth as a prior for sparse-view radiance field training

> [!needs-source] This thread is a scaffold. First ingest will populate it.
