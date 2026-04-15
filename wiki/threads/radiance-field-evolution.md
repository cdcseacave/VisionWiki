---
title: Radiance Field Evolution
type: thread
tags: [nerf, 3dgs, differentiable-rendering, novel-view-synthesis, sparse-voxels]
created: 2026-04-11
updated: 2026-04-15
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

---

## Goal & success criteria

Produce a radiance field from posed images that balances **rendering quality** (PSNR / SSIM / LPIPS on Mip-NeRF 360 + T&T), **rendering speed** (FPS at 1080p), **training cost** (per-scene wall-clock), and **scalability** (memory at city-scale). No single component dominates all four; the thread tracks per-regime winners.

## Current SOTA pipelines (as of 2026-04-15)

**Object / room scale (PSNR-dominant)**: 3DGS primitives + MVS-depth geometry regularization + decoupled appearance module.
- Primitive: 3DGS (Kerbl 2023).
- Geometry regularization: multiview-depth + median-depth relative loss. Paper: [[kim2025_multiview-geometric-gs]].
- Appearance decoupling: per-image CNN transforming target, not representation. Paper: [[lin2024_vastgaussian]] (generalized from NeRF-W).
- SfM/pose source: COLMAP (or InstantSfM when speed matters); CamP preconditioning when poses are noisy. Paper: [[park2023_camp]].

**City / aerial scale**: 3DGS + spatial partitioning + decoupled appearance + optional compression.
- Partition: airspace-aware `m×n` ground-plane cells with visibility-based camera selection. Paper: [[lin2024_vastgaussian]].
- Compression: codebook vector quantization + tetrahedral-mesh init. Paper: [[guo2025_ea-3dgs]]. Orthogonal to partitioning; stackable.
- Drone specifics: adaptive local-global distractor masking + MVS-voxel-guided sparse-view regularization. Paper: [[tang2025_dronesplat]].

**NeRF-family alternative (reflective / unbounded quality ceiling)**: Zip-NeRF stack.
- Scene contraction + proposal network + distortion regularizer. Paper: [[barron2022_mip-nerf-360]].
- Hash-grid + multisample anti-aliasing + Z-aliasing normalization. Paper: [[barron2023_zip-nerf]].
- Camera preconditioning. Paper: [[park2023_camp]].

**Neural-free alternative**: sparse-voxel rasterization.
- Adaptive sparse voxels + Morton-ordered CUDA rasterizer + post-activation density. Paper: [[sun2025_sparse-voxels-rasterization]].
- Natively mesh-extractable (see [[gaussian-to-mesh-pipelines]] Paradigm C).

## Pipeline lineage

- 2020 · baseline: volumetric rendering via MLP. Driver: NeRF.
- 2022 · unbounded scenes: contract + proposal + distortion. Driver: [[barron2022_mip-nerf-360]].
- 2023 · speed: hash-grid + anti-aliased cone. Driver: [[barron2023_zip-nerf]].
- 2023 · implicit-NeRF ceiling: per-frame pose preconditioning. Driver: [[park2023_camp]].
- 2023 · representation switch: implicit MLP → explicit Gaussians. Driver: Kerbl (3DGS).
- 2024 · city scale: partition + airspace-aware visibility + 2D-transform appearance. Driver: [[lin2024_vastgaussian]].
- 2025 · geometry: external MVS depth + median-depth relative loss. Driver: [[kim2025_multiview-geometric-gs]].
- 2025 · compression: tetra-mesh init + codebook VQ. Driver: [[guo2025_ea-3dgs]].
- 2025 · aerial robustness: adaptive masking + MVS-voxel regularization. Driver: [[tang2025_dronesplat]].
- 2025 · alternative primitive: sparse voxel rasterizer. Driver: [[sun2025_sparse-voxels-rasterization]].
- 2026 · city-scale SLAM: progressive voxel-GS with submaps + loop closure. Driver: [[deng2026_vpgs-slam]].

## Candidate components / not yet integrated

- **Per-pixel confidence from CoMe** as an alternative to SSIM in VastGaussian's decoupled appearance module. Proposed in [[radl2026_confidence-mesh-3dgs]] discussion; no paper merges them.
- **Pow3R-style prior injection** (intrinsics / poses / sparse depth) into a 3DGS training frontend. No paper does this yet; 3DGS is still COLMAP-posed.
- **Feed-forward 3DGS initialization** (PixelSplat, MVSplat, NoPoSplat) replacing COLMAP-SfM in the per-scene pipeline.
- **Foundation-feature distillation into per-Gaussian features** — tracked under [[lifting-foundation-models-to-3d]] but not yet in any Current-SOTA radiance pipeline.

## Open questions & synthesis bets

- Is SVRaster's sparse-voxel approach a serious competitor to Gaussians? **Synthesis bet**: *SVRaster primitives + CoMe confidence + MVS-depth supervision*. Combines [[sun2025_sparse-voxels-rasterization]] + [[radl2026_confidence-mesh-3dgs]] + [[schonberger2016_colmap-mvs]]. Could beat all 3DGS-based pipelines on geometry without losing render speed.
- City-scale (km²): VastGaussian for partitioning, EA-3DGS for compression, VPGS-SLAM for incremental capture. **Synthesis bet**: *all three layered* — partition at capture time (VPGS-SLAM submaps), compress per submap (EA-3DGS codebooks), merge at render time. No paper composes the three.
- Can CamP's preconditioning idea port to 3DGS? Paper explicitly leaves it open. **Synthesis bet**: *CamP-style Jacobian preconditioner on the 3DGS pose-refinement residual* (when poses are jointly optimized with Gaussians). Generic preconditioning is representation-agnostic; needs a 3DGS paper to try.

## Contradictions & tensions

- "External MVS depth > self-supervised 3DGS depth" (Kim 2025) vs. "self-supervised CoMe confidence wins" (Radl 2026) — same contradiction as in [[gaussian-to-mesh-pipelines]]. Open.
- Z-aliasing fix in Zip-NeRF is empirical with no principled explanation. No follow-up paper has closed this.

