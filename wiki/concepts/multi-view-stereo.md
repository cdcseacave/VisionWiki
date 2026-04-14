---
title: Multi-View Stereo
type: concept
tags: [mvs, depth, dense-reconstruction]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/schonberger2016_colmap-mvs.md, wiki/papers/yao2018_mvsnet.md]
status: draft
---

## What it is

Multi-View Stereo (MVS) produces a **dense 3D reconstruction** (depth maps, point clouds, or meshes) from a set of images with *known* camera poses — typically supplied by [[structure-from-motion]].

## Core approaches

- **PatchMatch MVS** — per-pixel depth hypotheses propagated across neighbors (COLMAP, OpenMVS).
- **Semi-global matching (SGM)** — pathwise cost aggregation for stereo / rectified MVS.
- **Learned MVS** — cost-volume networks (MVSNet lineage), foundation-feature MVS.
- **Feed-forward geometry** — methods like [[dust3r|DUSt3R]] / [[mast3r|MASt3R]] blur the SfM/MVS boundary by emitting dense pointmaps directly.

## Outputs
- Depth / disparity maps per view.
- Fused point clouds.
- Downstream meshing via Poisson, TSDF fusion, or neural surfaces ([[signed-distance-field|SDF]]).

## Relation to radiance fields
Radiance-field methods (NeRF, 3DGS) are an implicit form of MVS: dense view-consistent geometry emerges from photometric supervision rather than explicit stereo matching.

## Key references
- [Schönberger et al. 2016](../papers/schonberger2016_colmap-mvs.md) · [pdf](../../papers/mvs-depth/schonberger_2016_colmap-mvs.pdf) — COLMAP MVS with pixelwise view selection.
- [Yao et al. 2018](../papers/yao2018_mvsnet.md) · [pdf](../../papers/mvs-depth/yao_2018_mvsnet.pdf) — MVSNet, the canonical learned MVS architecture.
