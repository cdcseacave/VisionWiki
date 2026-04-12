---
title: Gaussian-to-Mesh Pipelines
type: thread
tags: [3dgs, mesh-reconstruction, surface-extraction, marching-cubes]
created: 2026-04-12
updated: 2026-04-12
sources: []
status: stub
---

## Working hypothesis
_(to be filled as papers are ingested)_

3D Gaussian Splatting produces high-quality novel views but the underlying
representation — a cloud of anisotropic Gaussians — is not directly usable
in traditional geometry pipelines that expect meshes. This thread tracks the
emerging approaches for bridging that gap:

1. Direct surface extraction from Gaussians (opacity thresholding, SDF fitting).
2. Hybrid representations that maintain Gaussian rendering quality while being
   mesh-convertible (2DGS, SuGaR, GOF, etc.).
3. Post-hoc conversion: Poisson/TSDF on Gaussian centers vs. learned extractors.
4. Quality-fidelity tradeoffs: what you lose going from splats to triangles.
5. Downstream usability: texturing, editing, physics simulation, game engines.

## Evidence
_(empty — will accumulate as papers are ingested)_

## Open questions
- Is there a representation that matches 3DGS rendering quality *and* gives
  watertight meshes without a separate extraction step?
- How do mesh extraction methods compare under sparse-view regimes vs dense capture?
- What's the practical path from a phone scan → Gaussian splat → textured mesh
  ready for a game engine?

## Related threads
- [[radiance-field-evolution]]
