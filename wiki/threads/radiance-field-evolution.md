---
title: Radiance Field Evolution
type: thread
tags: [nerf, 3dgs, differentiable-rendering, novel-view-synthesis, sparse-voxels]
created: 2026-04-11
updated: 2026-04-14
sources: [papers/park2023_camp.md, papers/xie2025_gauss-mi.md, papers/tang2025_dronesplat.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md, papers/kim2025_multiview-geometric-gs.md, papers/guo2025_ea-3dgs.md, papers/deng2026_vpgs-slam.md, papers/lin2024_vastgaussian.md, papers/barron2022_mip-nerf-360.md, papers/barron2023_zip-nerf.md]
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
- [Mip-NeRF 360 (Barron 2022)](../papers/barron2022_mip-nerf-360.md) is the
  last implicit-NeRF paper to materially move the frontier. Three coordinated
  tricks — a non-linear scene contraction for unbounded captures, a
  proposal-network-as-online-distillation for cheap importance sampling,
  and a first-principles **distortion regularizer** against floaters and
  background collapse — produce 57% lower MSE than Mip-NeRF, and the
  dataset it introduced became the canonical outdoor-NVS benchmark for
  every subsequent radiance-field paper.
- [Zip-NeRF (Barron 2023)](../papers/barron2023_zip-nerf.md) was the
  high-water-mark of implicit radiance fields: fuses Mip-NeRF 360's
  contraction + proposal network with Instant-NGP's hash-grid speed,
  achieving **8× lower error than Mip-NeRF 360** at ~1h training. Its
  existence made the NeRF→3DGS hand-off a fair fight — 3DGS won not on
  raw quality but on *edit-ability* and real-time rendering, not on a
  weak NeRF baseline.
- [CamP (Park 2023)](../papers/park2023_camp.md) showed that NeRF's joint
  pose-radiance optimization is brittle due to ill-conditioned camera
  parameterization. Camera preconditioning via whitening reduced RMSE by 67%
  (stacked on Zip-NeRF) — but this is a patch on an inherently fragile
  pipeline, and no 3DGS paper in this batch has bothered replicating the
  trick. That's a tell: when pose jitter matters enough to invent CamP, the
  field is already shopping for a more robust representation.
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
Three distinct attacks on the same problem — they are orthogonal and could be
stacked.

- **Spatial partitioning.** [VastGaussian (Lin 2024)](../papers/lin2024_vastgaussian.md)
  — the *first* real-time 3DGS method for aerial / city-scale scenes. Ground-plane
  `m × n` partition with airspace-aware visibility (floaters live off the
  surface, so visibility must cover the full vertical column, not just the
  surface convex hull). Per-cell parallel training + seamless merge matches or
  beats Mega-NeRF/Switch-NeRF on SSIM and LPIPS *every scene*, at 170+ FPS
  and 10× shorter training. Decoupled appearance modeling (per-image CNN
  transforming the rendered target, not the representation) is the
  rasterization-compatible successor to NeRF-W's GLO embeddings.
- **Compression.** [EA-3DGS (Guo 2025)](../papers/guo2025_ea-3dgs.md):
  tetrahedral mesh init + codebook quantization achieves 5.2x compression for
  outdoor scenes.
- **Incremental SLAM.** [VPGS-SLAM (Deng 2026)](../papers/deng2026_vpgs-slam.md):
  voxel-based progressive 3DGS SLAM handles large-scale indoor/outdoor with loop
  closure, avoiding a priori partitioning.

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
- ~~Can 3DGS scale to city-scale reconstruction without quantization/LOD tricks?~~
  Partially answered: VastGaussian scales to aerial UrbanScene3D captures via
  spatial partitioning alone (no quantization), beating NeRF-based large-scene
  methods on quality and speed. *Open*: true city-scale (km²) likely still needs
  partitioning + compression layered together; no automatic cell-count
  selection exists.
- Can the decoupled appearance-modeling trick (train-time 2D transform,
  discarded at inference) be ported to *other* rasterization-based renderers
  (SVRaster, neural primitives) as a general recipe against photometric drift?

## Related threads
- [[gaussian-to-mesh-pipelines]] — the downstream question of what to do with Gaussians
- [[feed-forward-structure-from-motion]] — feed-forward methods that may replace the SfM stage radiance fields depend on
- [[mono-depth-estimation]] — mono depth as a prior for sparse-view radiance field training

