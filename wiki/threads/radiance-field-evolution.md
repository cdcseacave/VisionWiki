---
title: Radiance Field Evolution
type: thread
tags: [nerf, 3dgs, differentiable-rendering, novel-view-synthesis, sparse-voxels]
created: 2026-04-11
updated: 2026-04-12
sources: [papers/park2023_camp.md, papers/xie2025_gauss-mi.md, papers/tang2025_dronesplat.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md, papers/kim2025_multiview-geometric-gs.md, papers/guo2025_ea-3dgs.md, papers/deng2026_vpgs-slam.md]
status: draft
---

## Working hypothesis

The field has moved decisively from implicit MLPs (NeRF) to explicit primitives
(3DGS), with the current frontier being: (a) how to get geometry quality to
match or exceed neural implicit surfaces, and (b) how to scale to large outdoor
scenes without blowing up memory. A third competing direction — sparse voxel
rasterization — sidesteps Gaussians entirely while matching their speed.

## Evidence

### The NeRF → 3DGS transition
- [CamP (Park 2023)](../papers/park2023_camp.md) showed that NeRF's joint
  pose-radiance optimization is brittle due to ill-conditioned camera
  parameterization. Camera preconditioning via whitening reduced RMSE by 67% —
  but this is a patch on an inherently fragile pipeline.
- The 3DGS papers in this batch (6 of 7 radiance-field papers) all build on
  [[3d-gaussian-splatting]] as the base representation, confirming its dominance.

### Quality frontier: geometry from Gaussians
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md): MVS depth + normal
  regularization pushes 3DGS to SOTA explicit surface reconstruction on DTU/TNT.
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md): per-Gaussian SDF values
  enable relightable inverse rendering — bridging appearance and geometry without
  separate neural fields.
- [DroneSplat (Tang 2025)](../papers/tang2025_dronesplat.md): MVS-guided voxel
  optimization + distractor masking makes 3DGS robust under drone imagery
  (sparse views, dynamic objects, lighting variation).

### Scaling: outdoor and large scenes
- [EA-3DGS (Guo 2025)](../papers/guo2025_ea-3dgs.md): tetrahedral mesh init +
  codebook quantization achieves 5.2x compression for outdoor scenes.
- [VPGS-SLAM (Deng 2026)](../papers/deng2026_vpgs-slam.md): voxel-based
  progressive 3DGS SLAM handles large-scale indoor/outdoor with loop closure.

### Alternative representations
- [SVRaster (Sun 2025)](../papers/sun2025_sparse-voxels-rasterization.md):
  sparse voxel rasterization with Morton ordering matches 3DGS quality/speed
  *without Gaussians at all* — neural-free, artifact-free, and naturally
  produces meshes via [[marching-cubes]]. This may be a dark horse.

### Active reconstruction
- [GauSS-MI (Xie 2025)](../papers/xie2025_gauss-mi.md): real-time mutual
  information from 3DGS for next-best-view selection — showing 3DGS is mature
  enough to serve as a live reconstruction backend, not just offline.

## Open questions
- Which representation wins for *editable* geometry vs pure novel-view synthesis?
- ~~How does 3DGS composition interact with traditional bundle adjustment?~~
  Partially answered: CamP shows preconditioning helps NeRF; 3DGS papers
  mostly rely on COLMAP poses. Direct 3DGS+BA co-optimization still rare.
- What's the current best path from a Gaussian splat to a usable textured mesh?
  → See [[gaussian-to-mesh-pipelines]] — now has 7 papers addressing this.
- Is SVRaster's sparse voxel approach a serious competitor to Gaussians, or
  a niche alternative?
- Can 3DGS scale to city-scale reconstruction without quantization/LOD tricks?

## Related threads
- [[gaussian-to-mesh-pipelines]] — the downstream question of what to do with Gaussians
- [[feed-forward-structure-from-motion]] — feed-forward methods that may replace the SfM stage radiance fields depend on
- [[mono-depth-estimation]] — mono depth as a prior for sparse-view radiance field training
