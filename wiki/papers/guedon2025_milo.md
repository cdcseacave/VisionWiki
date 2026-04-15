---
title: "MILo: Mesh-In-the-Loop Gaussian Splatting for Detailed and Efficient Surface Reconstruction"
type: paper
tags: [gaussian-splatting, mesh-extraction, delaunay-triangulation, surface-reconstruction, differentiable-rendering]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/guedon_2025_milo.pdf
url: https://arxiv.org/abs/2506.24096
code: https://github.com/Anttwo/MILo
license_code: Gaussian-Splatting-License
license_paper: arxiv-nonexclusive
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/guedon_2025_milo.pdf) · [arXiv](https://arxiv.org/abs/2506.24096) · [code](https://github.com/Anttwo/MILo)

_Code license: `Gaussian-Splatting-License`_

## TL;DR

MILo introduces a fully differentiable mesh-in-the-loop framework for [[3d-gaussian-splatting]] that constructs a mesh at every training iteration directly from Gaussian parameters via Delaunay triangulation. This produces compact, high-fidelity meshes with an order of magnitude fewer vertices than prior methods, while achieving state-of-the-art surface reconstruction quality on Tanks and Temples and competitive results on DTU.

## Problem

Current Gaussian Splatting methods extract surfaces through costly post-processing steps (e.g., [[tsdf|TSDF]] fusion, [[marching-cubes]]), resulting in loss of fine geometric details or very dense meshes with millions of vertices. The a posteriori conversion from volumetric to surface representation fundamentally limits the final mesh's ability to preserve all geometric structures captured during training. Existing meshes are also too heavy for practical downstream applications like physics simulation and animation.

## Method

MILo introduces three key technical contributions:

1. **Bidirectional Consistency Framework**: Ensures both Gaussians and the extracted mesh capture the same underlying geometry during training. Gradient backpropagation flows from mesh losses directly to Gaussian parameters.

2. **Adaptive Mesh Extraction via Gaussian Pivots**: At each training iteration, uses Gaussians as differentiable pivots for [[delaunay-triangulation]]. The mesh vertices and connectivity are constructed directly from Gaussian parameters, enabling end-to-end differentiable mesh optimization.

3. **SDF from Gaussians**: A novel method for computing signed distance values from 3D Gaussians that enables precise surface extraction while avoiding geometric erosion.

The method works with any Gaussian splatting backend (demonstrated with RaDe-GS rasterizer). The base model uses 0.1--0.5M Gaussians (up to 10x fewer than competing methods), with a dense variant using 2--4M.

## Results

- **Tanks and Temples**: Best F1-score among all Gaussian-based approaches (dense variant). Achieves high quality with significantly fewer mesh vertices and lower resource requirements.
- **DTU**: Competitive Chamfer distance with significantly fewer Gaussians and mesh vertices compared to previous approaches.
- **Mip-NeRF 360 / Deep Blending**: Meshes produce clean, accurate results well-suited for downstream applications.
- **Efficiency**: Base model requires only 10GB VRAM; dense model up to 17GB on a single RTX 4090.
- Mesh-Based Novel View Synthesis metric on T&T shows strong correlation with F1-score.

## Why it matters

MILo is a conceptual advance: instead of extracting meshes as a post-processing afterthought, it makes mesh quality a first-class optimization objective during training. The resulting meshes are dramatically lighter (10x fewer vertices) while being more accurate, making them practical for downstream applications like physics simulation, animation, and real-time rendering. This bridges the gap between volumetric and surface representations.

## Pipeline contribution

- **Delaunay triangulation with Gaussians as differentiable pivots (N1)** — mesh reconstructed at every training step from Gaussian parameters. candidate thread: [[gaussian-to-mesh-pipelines]] Paradigm B · stage: *mesh-in-the-loop* · replaces/augments: *post-hoc TSDF + marching cubes* · expected gain: 10× fewer vertices at comparable/better accuracy; directly optimizes final mesh rather than a volumetric proxy.
- **Bidirectional consistency framework (N2)** — mesh losses backprop to Gaussian parameters. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *loss coupling* · expected gain: ensures Gaussians and mesh encode the same surface; no drift between the two representations.
- **SDF-from-Gaussians computation (N3)** — signed distance from 3D Gaussians for precise extraction without erosion. candidate thread: [[gaussian-to-mesh-pipelines]] · stage: *SDF derivation* · expected gain: geometric precision without a separate SDF network.
- **Synthesis-bet candidate**: *MILo's Delaunay-in-loop with CoMe's per-Gaussian confidence as vertex weights* — the bet flagged in [[gaussian-to-mesh-pipelines]]. Mechanisms compatible; no paper does this.

## Relation to prior work

- Builds on [[3d-gaussian-splatting]], specifically using the RaDe-GS codebase.
- Contrasts with post-hoc extraction methods: [[tsdf|TSDF]] fusion (used by 2DGS, GOF, PGSR), [[marching-cubes]] (used by SOF, GOF), SuGaR's Poisson reconstruction.
- Uses [[delaunay-triangulation]] instead of [[marching-cubes]] or [[poisson-reconstruction]] for mesh generation.
- Compared against GOF, [[2d-gaussian-splatting]], PGSR, RaDe-GS, GS-Pull, VCR-GauS, Neuralangelo, NeuS, VolSDF.

## Open questions / limitations

- The dense variant requires more memory and training time.
- DTU evaluation focuses on small, object-centric scenes where the mesh-in-the-loop approach has less advantage over simpler methods.
- Background handling relies on a separate evaluation protocol (Mesh-Based NVS) since no dataset provides ground truth for backgrounds.

## Code & license

Inria/MPII Gaussian-Splatting License — non-commercial research use only (inherits 3DGS base terms). Blocks commercial integration without separate licensing from Inria.

## References added to the wiki

- [[[guedon2025_milo]]] (this page)
