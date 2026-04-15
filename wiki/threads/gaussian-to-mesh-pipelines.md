---
title: Gaussian-to-Mesh Pipelines
type: thread
tags: [3dgs, mesh-reconstruction, surface-extraction, marching-cubes, sdf, tsdf]
created: 2026-04-12
updated: 2026-04-15
sources: [papers/li2025_geosvr.md, papers/li2025_va-gs.md, papers/gao2025_anisdf.md, papers/radl2025_sof.md, papers/guedon2025_milo.md, papers/radl2026_confidence-mesh-3dgs.md, papers/elflein2026_vgg-t3.md, papers/kim2025_multiview-geometric-gs.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md, papers/chen2024_pgsr.md]
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
  Mesh via [[tsdf|TSDF]] fusion. Strong DTU results.
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
- [PGSR (Chen 2024)](../papers/chen2024_pgsr.md): flattens Gaussians into
  planes + unbiased-depth rendering + multi-view geometric regularization.
  Matches neural-SDF surface quality without external priors — a strong
  argument that the 3DGS↔surface gap is a regularization problem, not a
  representation problem. Natural fit for Paradigm A.

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

---

## Goal & success criteria

From posed images + 3DGS primitives (or alternative), produce a watertight textured mesh. "Better" = higher T&T F-score (Intermediate + Advanced) and lower DTU Chamfer + **reasonable training time** (< 1 h per scene for the lane that cares) + **no manual cleanup** + rendering quality preserved if the method claims a joint appearance+geometry representation.

## Current SOTA pipeline (as of 2026-04-15)

Three competing pipelines tracked; swap depends on which constraint dominates:

**Pipeline A — Regularized 3DGS → TSDF/MC** (most popular; best when rendering quality matters):
1. 3DGS training with geometric regularization (planar, normal, edge, visibility alignment losses). Paper: [chen2024_pgsr] (PGSR), [li2025_va-gs] (VA-GS).
2. External MVS depth supervision (via [schonberger2016_colmap-mvs] or learned MVS). Paper: [kim2025_multiview-geometric-gs].
3. Confidence-weighted depth rendering. Paper: [radl2026_confidence-mesh-3dgs] (CoMe) — current SOTA on T&T (F1 0.521).
4. TSDF fusion [curless1996_tsdf] + marching cubes.

**Pipeline B — Mesh-in-the-loop** (best when mesh size matters):
1. Delaunay triangulation at every step. Paper: [guedon2025_milo] (MILo). 10× fewer vertices; comparable/better geometry; 3× longer training than CoMe.

**Pipeline C — Natively mesh-extractable** (best when Gaussians are optional):
1. Sparse voxels with uncertainty-weighted depth. Paper: [li2025_geosvr] (SOTA DTU Chamfer) or [sun2025_sparse-voxels-rasterization] (SVRaster).
2. Direct marching cubes on the voxel/SDF field. No Gaussians.

## Pipeline lineage

- 1996 · surface fusion: ad-hoc range-image merging → TSDF weighted-average. Driver: [curless1996_tsdf].
- 2016 · MVS depth source: baseline PatchMatch → COLMAP MVS with per-pixel view selection. Driver: [schonberger2016_colmap-mvs].
- 2018 · MVS depth alternative: classical PatchMatch → learned plane-sweep. Driver: [yao2018_mvsnet].
- 2024 · 3DGS regularization: unregularized splatting → planar-constrained + unbiased depth. Driver: [chen2024_pgsr].
- 2025 · external vs. self-supervised depth: self-supervised wins visually, external MVS wins on geometry. Driver: [kim2025_multiview-geometric-gs] → formalized as "external MVS > Gaussian self-supervision for mesh quality".
- 2026 · confidence weighting: uniform fusion weight → per-Gaussian self-supervised confidence. Driver: [radl2026_confidence-mesh-3dgs].

## Candidate components / not yet integrated

- **TSDF fusion with per-Gaussian CoMe confidence as the weight** — the TSDF weighting slot exists since 1996; CoMe confidence exists since 2026; no paper combines them. Blocked on: implementation, not novelty.
- **MVSNet-family learned MVS as the depth source** for Pipeline A — every current Pipeline A paper uses classical COLMAP MVS or self-supervised depth. Blocked on: speed/quality comparison.
- **SAM 3 masks as geometry constraints** — on-surface vs. off-surface Gaussian supervision from segmentation. Blocked on: no paper.

## Open questions & synthesis bets

- Watertight mesh in one step without a separate extraction: MILo comes closest. **Synthesis bet 1**: *combine CoMe's confidence + MILo's Delaunay-in-the-loop — use confidence as Delaunay vertex weighting*. Mixes [radl2026_confidence-mesh-3dgs] + [guedon2025_milo]. Expected: MILo's 10× vertex reduction with CoMe's 3× speedup.
- **Synthesis bet 2**: *Pipeline A with [schonberger2016_colmap-mvs]-quality depth used as the supervision source inside a CoMe-trained 3DGS, with PGSR's planar constraint and VA-GS's visibility loss active simultaneously*. All four mechanisms exist; no paper stacks them. Expected: additional +0.02–0.05 F1 on T&T Advanced.
- **Synthesis bet 3**: *GeoSVR-style sparse voxels with per-Gaussian-equivalent confidence per voxel + external MVS depth supervision*. Mixes [li2025_geosvr] + [kim2025_multiview-geometric-gs] + [radl2026_confidence-mesh-3dgs]. Could put Pipeline C on top of Paradigm A's benchmarks.
- **Texturing** is still underaddressed — all three pipelines stop at geometry. Bet: *render-then-unwrap from 3DGS/SVRaster with DINOv3 dense features as a seam-minimizer*.

## Contradictions & tensions

- "External MVS depth > Gaussian self-supervision" (Kim 2025) conflicts with "Gaussian self-supervised confidence outperforms external supervision" (CoMe 2026). Resolution hypothesis: external depth wins when MVS quality is high (controlled capture); self-supervised wins when MVS is unreliable (sparse views, textureless surfaces). Open: nobody has a principled switch.

