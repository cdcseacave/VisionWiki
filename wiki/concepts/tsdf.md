---
title: Truncated Signed Distance Field (TSDF)
type: concept
tags: [sdf, volumetric, fusion, meshing]
created: 2026-04-14
updated: 2026-04-14
sources: [wiki/papers/curless1996_tsdf.md]
status: draft
---

## What it is

A **TSDF** is a voxel grid storing a signed distance to the nearest surface, *truncated* to a band near the zero-level set. It is the canonical representation for fusing multiple depth maps into a coherent surface.

## How it works
- Each voxel accumulates a running weighted average of signed distances from observed depth maps.
- The zero-crossing is extracted as a mesh via [[marching-cubes]].
- Truncation bounds memory and damps contributions far from the surface.

## Relation to other representations
- Dense counterpart to a neural [[signed-distance-field|SDF]] — explicit grid vs. implicit network.
- Input to KinectFusion-style real-time reconstruction and most classical RGB-D fusion pipelines.
- Used in 3DGS-to-mesh pipelines (see [[gaussian-to-mesh-pipelines]]) to extract surfaces from rendered depths.
- Substrate for voxel-based foundation-model lifting: [[jatavallabhula2023_conceptfusion|ConceptFusion]] augments each voxel with CLIP/DINO/AudioCLIP features alongside the signed distance (see [[lifting-foundation-models-to-3d]]).

## Strengths / limitations
- **+** Simple, parallelizable, well-understood.
- **−** Memory scales cubically; fine detail requires hierarchical voxel structures (VDB, octrees).

## Key references
- [Curless & Levoy 1996](../papers/curless1996_tsdf.md) · [pdf](../../papers/mesh-reconstruction/curless_1996_tsdf.pdf) — SIGGRAPH; the original TSDF formulation.
- Newcombe et al. 2011, *KinectFusion: Real-Time Dense Surface Mapping and Tracking* — ISMAR; the real-time GPU-TSDF pipeline. No arXiv preprint.

> [!needs-source] — KinectFusion not yet ingested.
