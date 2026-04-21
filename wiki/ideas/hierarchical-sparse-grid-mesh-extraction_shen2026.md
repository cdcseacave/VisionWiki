---
title: Hierarchical sparse-grid mesh extraction for large-scale scenes (OpenVDB + per-level marching cubes)
type: idea
source_paper: wiki/papers/shen2026_lyra2.md
also_in: []

scope: stage-swap
stages: [mesh-reconstruction.extraction]
collapses: []
splits_into: []
rewrites: {}

inputs: [3dgs-scene, per-gaussian-depth]
outputs: [triangle-mesh-at-scene-scale]
assumptions: [3dgs-has-per-gaussian-depth-signal, scene-extent-estimable-for-level-allocation, signed-distance-field-derivable-from-depth]
requires_upstream_property: [3dgs-with-reliable-depth-per-gaussian]
requires_downstream_property: [none]
learned_params: []
failure_modes: [level-boundary-seam-artifacts-at-transitions, scale-parameter-sensitivity-in-level-allocation, no-isolated-ablation-in-source-paper]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mesh-reconstruction, mesh-extraction, openvdb, marching-cubes, sparse-grid, large-scale, lyra-2]
created: 2026-04-21
updated: 2026-04-21
status: unclaimed
validated_in: []
---

## Mechanism

Standard mesh extraction from 3DGS uses TSDF fusion over a dense voxel grid + marching cubes — fine for object-scale but catastrophic on city-scale or long-walkthrough scenes where a single uniform grid's memory scales with scene extent times inverse voxel-size cubed. Single-level sparse grids help but cannot simultaneously represent fine surface detail and long-range topology.

**The hierarchical sparse-grid recipe** (§4.4, App. A.4):

1. **Allocate a hierarchical OpenVDB [72, 115] sparse grid** with multiple levels; number of levels and per-level voxel sizes are determined by **the scale of the scene** (adaptive allocation — fine cells near generated content, coarse cells where needed for topology).
2. **Compute a signed distance field** from the 3DGS-derived per-Gaussian depth.
3. **Run marching cubes on each level** independently.
4. **Merge the per-level meshes at level transitions**, producing a single triangular surface mesh of the full scene.

OpenVDB is the reference library; the approach is specifically the *adaptive-level allocation + per-level MC + transition-merge* combination rather than a new marching cubes variant.

## Why it wins

**No isolated ablation in the paper** — this is flagged evidence weakness. The mesh-extraction step is described in App. A.4 as implementation detail supporting the applications section; the paper does not benchmark against alternative extraction pipelines.

**Design argument**: large-scale explorable scenes (Lyra 2.0's target) routinely span 100m+ with fine interior structure (tables, doorways). Uniform grids break at the extreme; single-level sparse grids must compromise somewhere. Hierarchical allocation is the principled fix and is supported by the applied literature (OpenVDB is the industry-standard sparse-grid datastructure for exactly this reason).

**Integration demonstration**: Fig. 7 shows the resulting mesh integrated into NVIDIA Isaac Sim for robot simulation — evidence that the geometry is watertight enough for physics engines without manual cleanup. Qualitative, not quantitative.

## Preconditions & compatibility

- **Upstream**: requires a 3DGS representation from which per-Gaussian depth can be converted to an SDF. Compatible with any 3DGS-producing pipeline (per-scene optimized or feed-forward).
- **Downstream**: produces a standard triangular mesh — any downstream mesh consumer works.
- **Alternative mesh-extraction stage fillers** currently in the wiki:
  - TSDF fusion + marching cubes (baseline in [[guedon2025_milo|MILo]], [[li2025_va-gs|VA-GS]], [[kim2025_multiview-geometric-gs|Kim 2025]])
  - Delaunay mesh-in-the-loop ([[delaunay-mesh-in-loop_guedon2025]])
  - Direct marching cubes on sparse voxels ([[li2025_geosvr|GeoSVR]], [[sun2025_sparse-voxels-rasterization|SVRaster]])
  - Confidence-weighted mesh extraction ([[radl2026_confidence-mesh-3dgs|CoMe]])

  The hierarchical-sparse-grid extraction competes on **scale** — none of the above are demonstrated at the scene sizes Lyra 2.0 targets (multi-room explorable walkthroughs). Which mechanism wins at scale is an open empirical question.

## Trade-offs vs. the decomposed pipeline

Not applicable — `scope: stage-swap` at `[[mesh-reconstruction.extraction]]`.

## Open questions

- **Quantitative comparison vs TSDF or MILo at scale**: the paper does not present head-to-head numbers (mesh quality, extraction time, vertex count) against other extraction pipelines at the same scene scale. This is the key empirical gap.
- **Level-boundary seams**: merging meshes at level transitions is a classical source of seam artifacts. The paper does not characterize seam frequency or provide a seam-repair step.
- **Automated level allocation**: "number of levels and voxel sizes for each level in the hierarchy are determined by the scale of the scene" — the exact heuristic is not spelled out. Reproducibility concern; may require per-scene tuning.
- **Transferability to non-3DGS representations**: the SDF-from-depth step assumes a 3DGS with per-Gaussian depth. For sparse-voxel representations (SVRaster, GeoSVR) the SDF is native; integration is open.
