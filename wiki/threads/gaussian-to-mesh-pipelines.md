---
title: Gaussian-to-Mesh Pipelines
type: thread
tags: [3dgs, mesh-reconstruction, surface-extraction, marching-cubes, sdf, tsdf]
created: 2026-04-12
updated: 2026-04-12
sources: [papers/li2025_geosvr.md, papers/li2025_va-gs.md, papers/gao2025_anisdf.md, papers/radl2025_sof.md, papers/guedon2025_milo.md, papers/radl2026_confidence-mesh-3dgs.md, papers/elflein2026_vgg-t3.md, papers/kim2025_multiview-geometric-gs.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md]
status: draft
---

## Working hypothesis

The Gaussian-to-mesh problem has split into three competing paradigms:
**(A)** regularize 3DGS during training so Gaussians align with surfaces, then
extract via TSDF/marching-cubes; **(B)** co-optimize a mesh jointly with
Gaussians (mesh-in-the-loop); **(C)** bypass Gaussians entirely with
representations that are natively mesh-extractable (sparse voxels, SDF fields).
Paradigm A is the most popular (5+ papers), B is the most principled (MILo),
and C may win long-term (SVRaster, GeoSVR).

## Evidence

### Paradigm A: Regularize Gaussians → post-hoc mesh extraction

- [VA-GS (Li 2025)](../papers/li2025_va-gs.md): four alignment losses
  (edge-aware, visibility-aware, normal, feature) force Gaussians onto surfaces.
  Mesh via [[TSDF]] fusion. Strong DTU results.
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md): MVS depth supervision
  + median-depth relative loss + normal/distortion regularization → SOTA explicit
  surface on DTU/TNT. Key insight: external MVS depth is more reliable than
  Gaussian self-supervision for geometry.
- [CoMe (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md): self-supervised
  per-Gaussian confidence scores balance photometric vs geometric loss +
  SSIM-decoupled appearance model. ~20 min runtime (vs MILo 60 min), SOTA on
  Tanks & Temples (F1 0.521) and ScanNet++.
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md): per-Gaussian discretized
  SDF values with learned SDF-to-opacity mapping. Enables both relighting and
  mesh extraction from the same representation.

### Paradigm B: Mesh-in-the-loop during training

- [MILo (Guedon 2025)](../papers/guedon2025_milo.md): Delaunay triangulation at
  every training step, differentiable mesh rendering in the loop. Result: 10x
  fewer mesh vertices than TSDF-based approaches, mesh quality comparable or
  better. Downside: more complex training pipeline.

### Paradigm C: Natively mesh-extractable representations

- [GeoSVR (Li 2025)](../papers/li2025_geosvr.md): explicit sparse voxels with
  uncertainty-weighted depth → direct [[marching-cubes]] extraction. No
  Gaussians at all. SOTA on DTU Chamfer distance.
- [SOF (Radl 2025)](../papers/radl2025_sof.md): sorted opacity fields with
  hierarchical resorting → 3x faster optimization, 10x faster meshing than
  standard 3DGS mesh pipelines. Robust marching-cubes from opacity fields.
- [SVRaster (Sun 2025)](../papers/sun2025_sparse-voxels-rasterization.md):
  sparse voxel rasterization is neural-free and produces meshes via standard
  marching-cubes. Matches 3DGS rendering quality.
- [AniSDF (Gao 2025)](../papers/gao2025_anisdf.md): fused-granularity SDF with
  anisotropic spherical Gaussians. Pure SDF approach — meshes come free via
  marching-cubes. Excels on reflective objects.

### Paradigm D: Feed-forward mesh prediction

- [VGG-T3 (Elflein 2026)](../papers/elflein2026_vgg-t3.md): replaces KV
  attention with test-time-trained MLPs for O(n) feed-forward 3D reconstruction.
  Outputs mesh/occupancy directly — no per-scene optimization needed.

## Emerging patterns

- **TSDF + marching-cubes** remains the dominant extraction path (used by VA-GS,
  Kim, SOF, and as a baseline in MILo).
- **Depth supervision from external MVS** consistently outperforms Gaussian
  self-supervision for geometry (Kim 2025, DroneSplat).
- **Confidence/uncertainty** is a recurring theme: Radl 2026 (per-Gaussian
  confidence), GeoSVR (voxel uncertainty), MILo (rendering confidence).

## Open questions
- Is there a representation that matches 3DGS rendering quality *and* gives
  watertight meshes without a separate extraction step? MILo comes closest
  but the training cost is higher.
- How do these methods compare under sparse-view regimes vs dense capture?
  Most benchmark on DTU (49 views) — few test at 3-10 views.
- What's the practical path from phone scan → splat → textured mesh for a
  game engine? Texturing is under-addressed — most papers stop at geometry.
- Will sparse voxels (GeoSVR, SVRaster) displace Gaussians for applications
  that need meshes?

## Related threads
- [[radiance-field-evolution]]
- [[feed-forward-structure-from-motion]] — VGG-T3 connects feed-forward 3D to mesh output
- [[mono-depth-estimation]] — mono depth priors improve Gaussian geometry
- [[nerfstudio]] — codebase map for the local 3DGS implementation

## Implementation notes
- [[come-integration-nerfstudio]] — design doc for porting CoMe into the local visiofacto fork
