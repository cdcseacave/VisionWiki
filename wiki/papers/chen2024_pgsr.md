---
title: "PGSR: Planar-based Gaussian Splatting for Efficient and High-Fidelity Surface Reconstruction"
type: paper
tags: [3dgs, surface-reconstruction, planar-primitives, mesh]
created: 2026-04-15
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/chen_2024_pgsr.pdf
url: https://arxiv.org/abs/2406.06521
status: stable
---

📄 [Full paper](../../papers/mesh-reconstruction/chen_2024_pgsr.pdf) · [arXiv](https://arxiv.org/abs/2406.06521)

## TL;DR

Chen et al. (Zhejiang / SenseTime, TVCG 2024) present **PGSR**, a [[3d-gaussian-splatting|3DGS]] variant where each Gaussian is regularized to be **effectively planar** and contributes an **unbiased depth** — enabling high-fidelity surface reconstruction directly from multi-view RGB **without** external depth or normal priors. Outputs meshes competitive with neural-SDF methods at 3DGS speeds.

## Problem

Vanilla 3DGS excels at rendering but produces **noisy, inconsistent surfaces**: the volumetric Gaussian representation does not commit to a specific surface location, so meshing via TSDF or Poisson on rendered depths gives soft, fuzzy results. Prior 3DGS-to-surface methods (SuGaR, 2DGS) improved on this but still lagged neural-SDF methods like NeuS/VolSDF, and/or required external monocular priors.

## Method

- **Flatten Gaussians into planes**: heavily penalize the smallest-eigenvalue scale → each Gaussian approximates an oriented disk with a well-defined normal.
- **Unbiased depth rendering**: derive depth analytically as (distance-to-plane) / (dot product of view direction with plane normal), eliminating the bias that standard α-composited depth introduces for anisotropic Gaussians.
- **Single-view geometric regularization**: rendered normals + depth should be locally consistent (co-planar neighbors).
- **Multi-view geometric regularization**: cross-view photometric + geometric consistency (à la classical PatchMatch MVS) pulls Gaussian planes onto the true surface.
- **No priors needed**: unlike many competitors, PGSR does not require pretrained monocular depth or normal networks.

## Results

- SOTA surface quality on DTU, Tanks-and-Temples, and Mip-NeRF 360 among 3DGS-based methods.
- Matches or beats neural-SDF methods (NeuS, Neuralangelo) on geometric accuracy — with dramatically lower training time.
- Rendering quality remains close to vanilla 3DGS.

## Why it matters

Strong evidence that the 3DGS ↔ surface gap is a **regularization problem, not a representation problem** — explicit planar constraints + unbiased depth close most of the distance to neural-SDF quality while preserving 3DGS's speed. Natural fit for the [[gaussian-to-mesh-pipelines]] thread.

## Pipeline contribution

- **Flattened-Gaussian constraint + unbiased depth rendering (N1)** — smallest-eigenvalue penalty → oriented disks; depth = (dist-to-plane)/(view-dir · normal). candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm A · stage: *primitive geometry constraint* · replaces/augments: *α-composited depth (biased)* · expected gain: closes most of the gap to neural-SDF methods without external priors; SOTA on DTU/T&T/Mip-360 among 3DGS surface methods.
- **Single-view geometric regularization (N2)** — rendered normals + depth co-planarity of neighbors. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *per-view loss* · expected gain: local surface smoothness.
- **Multi-view photometric + geometric consistency (N3)** — PatchMatch-style cross-view consistency. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *multi-view loss* · replaces/augments: *external MVS depth supervision (Kim 2025)* · expected gain: works without external priors.
- **Synthesis-bet contribution**: *PGSR's planar constraint + CoMe's confidence-weighted fusion + Kim 2025's MVS depth supervision* all apply to different sub-problems of the same pipeline. Stacked, they should surpass any individual component — this is the Pipeline-A composition bet formalized in [[gaussian-to-mesh-pipelines]].

## Relation to prior work

- Extends [[3d-gaussian-splatting]] with planarity + unbiased depth.
- Sibling to [[2d-gaussian-splatting|2DGS]] (Huang et al. 2024): 2DGS replaces the primitive, PGSR keeps 3D Gaussians but constrains them flat.
- Complements [[radl2025_sof|SOF]], [[li2025_geosvr|GeoSVR]], [[kim2025_multiview-geometric-gs|Multiview Geometric GS]] — all 3DGS-for-surfaces methods.

## Open questions / limitations

- Scene with genuinely thin/non-planar structure (hair, foliage) resists planar flattening — some fidelity loss there.
- Multi-view consistency term is expensive (re-projects + photometric checks) during training.

## References added to the wiki

- [[3d-gaussian-splatting]], [[gaussian-to-mesh-pipelines]] — cross-links + lineage.
