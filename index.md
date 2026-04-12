# Vision Wiki — Index

A content-oriented catalog of every page. Grouped by type.
For chronological activity, see [log.md](log.md). For the operating rules,
see [CLAUDE.md](CLAUDE.md).

## Papers

### SfM / SLAM
- [InstantSfM](wiki/papers/zhong2026_instantsfm.md) — GPU-native global SfM, 40x faster than COLMAP · _2026-04-12_
- [MP-SfM](wiki/papers/pataki2025_mp-sfm.md) — monocular depth/normal priors for robust incremental SfM · _2026-04-12_
- [VPGS-SLAM](wiki/papers/deng2026_vpgs-slam.md) — voxel-based progressive 3DGS SLAM for large scenes · _2026-04-12_
- [CuSfM](wiki/papers/yu2025_cusfm.md) — CUDA-accelerated SfM for autonomous driving · _2026-04-12_
- [MASt3R-SLAM](wiki/papers/murai2025_mast3r-slam.md) — real-time dense SLAM on MASt3R priors, 15 FPS · _2026-04-12_
- [MegaSaM](wiki/papers/li2025_megasam.md) — SfM from casual dynamic videos via motion segmentation · _2026-04-12_
- [DiffusionSfM](wiki/papers/zhao2025_diffusionsfm.md) — diffusion model for joint structure and motion prediction · _2026-04-12_
- [LoGeR](wiki/papers/zhang2025_loger.md) — long-context (19K frames) feedforward reconstruction via TTT · _2026-04-12_
- [ZipMap](wiki/papers/jin2026_zipmap.md) — linear-time bidirectional reconstruction, 20x faster than VGGT · _2026-04-12_

### Mesh Reconstruction
- [GeoSVR](wiki/papers/li2025_geosvr.md) — sparse voxels with uncertainty depth for SOTA surface reconstruction · _2026-04-12_
- [VA-GS](wiki/papers/li2025_va-gs.md) — multi-faceted view alignment for 3DGS surface quality · _2026-04-12_
- [AniSDF](wiki/papers/gao2025_anisdf.md) — fused-granularity SDF with anisotropic encoding for reflective objects · _2026-04-12_
- [SOF](wiki/papers/radl2025_sof.md) — sorted opacity fields, 3x faster optimization + 10x faster meshing · _2026-04-12_
- [MILo](wiki/papers/guedon2025_milo.md) — mesh-in-the-loop Gaussian splatting via Delaunay triangulation · _2026-04-12_
- [Confidence Mesh from 3DGS](wiki/papers/radl2026_confidence-mesh-3dgs.md) — per-Gaussian confidence for geometry/appearance balance · _2026-04-12_
- [VGG-T3](wiki/papers/elflein2026_vgg-t3.md) — O(n) feed-forward 3D reconstruction via test-time-trained MLPs · _2026-04-12_
- [SpatialLM](wiki/papers/mao2025_spatiallm.md) — LLM fine-tuned on point clouds for structured indoor modeling · _2026-04-12_

### Radiance Fields
- [GauSS-MI](wiki/papers/xie2025_gauss-mi.md) — real-time mutual information for active 3DGS view selection · _2026-04-12_
- [DroneSplat](wiki/papers/tang2025_dronesplat.md) — robust 3DGS from in-the-wild drone imagery · _2026-04-12_
- [CamP](wiki/papers/park2023_camp.md) — camera preconditioning for joint pose-NeRF optimization · _2026-04-12_
- [GS + Discretized SDF](wiki/papers/zhu2025_gs-discretized-sdf.md) — per-Gaussian SDF for relightable inverse rendering · _2026-04-12_
- [SVRaster](wiki/papers/sun2025_sparse-voxels-rasterization.md) — sparse voxel rasterization matching 3DGS quality, neural-free · _2026-04-12_
- [Multiview Geometric GS](wiki/papers/kim2025_multiview-geometric-gs.md) — MVS depth + normal regularization for SOTA explicit surfaces · _2026-04-12_
- [EA-3DGS](wiki/papers/guo2025_ea-3dgs.md) — efficient adaptive 3DGS with 5.2x compression for outdoor · _2026-04-12_

### MVS / Depth
- [Multi-view Dense Matching](wiki/papers/chebbi2025_multiview-dense-matching.md) — similarity learning extended to multi-view without retraining · _2026-04-12_
- [Pow3R](wiki/papers/jang2025_pow3r.md) — conditioning DUSt3R with camera priors for native-resolution 3D · _2026-04-12_

### Pose Estimation
- [Cameras as Rays](wiki/papers/zhang2024_cameras-as-rays.md) — Plucker ray-bundle pose estimation via diffusion · _2026-04-12_

### Feature Matching
- [RoMa v2](wiki/papers/edstedt2025_roma-v2.md) — dense matcher with DINOv3 backbone, SOTA correspondence · _2026-04-12_

### Fundamentals
- [SD-RPN](wiki/papers/shi2026_self-distilled-roi.md) — self-distilled RoI predictors for fine-grained MLLM perception · _2026-04-12_
- [RADIOv2.5](wiki/papers/heinrich2025_radiov25.md) — agglomerative vision foundation model distilling CLIP+DINOv2+SAM · _2026-04-12_
- [Trident](wiki/papers/shi2024_open-vocab-segmentation.md) — training-free open vocabulary segmentation via CLIP+DINO+SAM · _2026-04-12_
- [Feed-Forward 3D Survey](wiki/papers/zhang2025_feed-forward-3d-survey.md) — comprehensive taxonomy of feed-forward 3D methods · _2026-04-12_

## Methods
- [3D Gaussian Splatting](wiki/methods/3d-gaussian-splatting.md) — explicit radiance field via anisotropic Gaussians · _stub_
- [2D Gaussian Splatting](wiki/methods/2d-gaussian-splatting.md) — flat-Gaussian variant for better surfaces · _stub_
- [NeRF](wiki/methods/nerf.md) — neural radiance fields via MLP + volume rendering · _stub_
- [COLMAP](wiki/methods/colmap.md) — the standard SfM/MVS pipeline · _stub_
- [DUSt3R](wiki/methods/dust3r.md) — feed-forward two-view 3D reconstruction · _stub_
- [MASt3R](wiki/methods/mast3r.md) — DUSt3R + dense matching · _stub_
- [VGGT](wiki/methods/vggt.md) — Visual Geometry Grounded Transformer · _stub_
- [DROID-SLAM](wiki/methods/droid-slam.md) — dense visual SLAM with differentiable BA · _stub_
- [Marching Cubes](wiki/methods/marching-cubes.md) — isosurface extraction from volumetric fields · _stub_

## Concepts
- [Bundle Adjustment](wiki/concepts/bundle-adjustment.md) — joint optimization of cameras + 3D points · _stub_
- [Signed Distance Field](wiki/concepts/signed-distance-field.md) — implicit surface representation · _stub_
- [Differentiable Rendering](wiki/concepts/differentiable-rendering.md) — gradient-based 3D optimization via rendering · _stub_
- [Test-Time Training](wiki/concepts/test-time-training.md) — adapting model weights at inference for long-context scaling · _stub_
- [Monocular Depth Estimation](wiki/concepts/monocular-depth-estimation.md) — single-image depth prediction · _stub_
- [Feature Matching](wiki/concepts/feature-matching.md) — correspondence finding between images · _stub_
- [Spherical Harmonics](wiki/concepts/spherical-harmonics.md) — basis functions for view-dependent color · _stub_
- [Vision Transformer](wiki/concepts/vision-transformer.md) — transformer architecture for visual tasks · _stub_

## Datasets
_(empty)_

## People
_(empty)_

## Threads
- [Radiance Field Evolution](wiki/threads/radiance-field-evolution.md) — NeRF → 3DGS lineage, representations, speed/quality tradeoffs · _updated 2026-04-12_
- [Gaussian-to-Mesh Pipelines](wiki/threads/gaussian-to-mesh-pipelines.md) — extracting usable meshes from Gaussian splats, 4 competing paradigms · _updated 2026-04-12_
- [Feed-Forward Structure from Motion](wiki/threads/feed-forward-structure-from-motion.md) — replacing iterative SfM: accelerated classical → hybrid → fully learned · _updated 2026-04-12_
- [Monocular Depth Estimation](wiki/threads/mono-depth-estimation.md) — mono depth as load-bearing prior for SfM/3DGS/MVS · _updated 2026-04-12_

---

_Last rebuilt: 2026-04-12 · 32 papers, 9 methods, 8 concepts, 4 threads_
