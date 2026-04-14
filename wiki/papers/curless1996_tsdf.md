---
title: "A Volumetric Method for Building Complex Models from Range Images"
type: paper
tags: [tsdf, volumetric, range-image, fusion, meshing]
created: 2026-04-14
updated: 2026-04-14
sources: []
local_paper: papers/mesh-reconstruction/curless_1996_tsdf.pdf
url: https://graphics.stanford.edu/papers/volrange/volrange.pdf
status: stable
---

📄 [Full paper](../../papers/mesh-reconstruction/curless_1996_tsdf.pdf) · [Stanford PDF](https://graphics.stanford.edu/papers/volrange/volrange.pdf)

## TL;DR

Curless & Levoy (SIGGRAPH 1996) introduce the **truncated signed distance field** ([[tsdf|TSDF]]): fuse multiple range images into a voxel grid storing a running weighted average of signed distances, then extract the surface at the zero-crossing via [[marching-cubes]]. Defines the modern volumetric-fusion template used by KinectFusion and essentially every RGB-D reconstruction system since.

## Problem

Prior range-image fusion methods (zippered polygon meshes, surface reconstruction from unorganized points) were brittle: sensitive to outliers, hole-filling was ad hoc, and fusing many overlapping scans produced geometric artifacts at seams. A principled incremental method was missing.

## Method

- **Per-voxel signed distance**: for each range image, compute the signed distance from each voxel to the nearest range sample along the sensor ray. Positive = in front of surface, negative = behind.
- **Truncation band**: signed distances only stored within a narrow band around the zero level set (bounded memory, localizes influence).
- **Incremental weighted average**: each voxel accumulates a running `(weighted_sum, weight_sum)` pair; new scans update it via confidence-weighted averaging. Handles arbitrary numbers of views without re-optimization.
- **Weights** encode sensor confidence (e.g. angle-of-incidence, range).
- **Hole filling**: explicit space-carving pass distinguishes unseen vs. empty space, enabling controlled hole closure.
- **Surface extraction**: [[marching-cubes]] on the zero-isosurface of the TSDF.

## Results

- Reconstructs the Stanford Bunny, Dragon, Buddha from hundreds of structured-light scans with topologically correct watertight meshes.
- Gracefully handles noise, grazing-angle data, and overlapping scans.

## Why it matters

TSDF is the **canonical bridge between depth observations and surfaces**. Downstream usage:
- KinectFusion (2011) and all real-time RGB-D SLAM variants.
- Voxel-hashing / VDB sparse variants for large scenes.
- Modern [[3d-gaussian-splatting|3DGS]]-to-mesh pipelines (see [[gaussian-to-mesh-pipelines]]): render depths, fuse into TSDF, extract via marching cubes.
- Dense counterpart to neural [[signed-distance-field|SDF]] representations — both target the same function, explicit vs. implicit.

## Relation to prior work

- Succeeds zippered meshes (Turk & Levoy 1994) as the Stanford-group fusion method.
- Precursor to Poisson Surface Reconstruction (Kazhdan 2006) on the "implicit-function" branch.
- Foundational to the KinectFusion (Newcombe 2011) real-time GPU pipeline — same math, GPU voxel grid, ICP-based pose tracking.

## Open questions / limitations

- Dense voxel grid scales cubically; hierarchical/sparse structures (octrees, VDB, voxel hashing) needed for large scenes.
- Weighted averaging assumes Gaussian noise — outliers still leak through at voxel level without explicit robust variants.
- No modeling of color/radiance — TSDF is geometry only.

## References added to the wiki

- [[tsdf]] concept — `[!needs-source]` resolved.
- [[marching-cubes]] method — TSDF is its most common input.
