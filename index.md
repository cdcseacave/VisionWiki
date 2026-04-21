# Vision Wiki — Index

A content-oriented catalog of every page. Grouped by type.
For chronological activity, see [log.md](log.md). For the operating rules,
see [CLAUDE.md](CLAUDE.md).

## Papers

### SfM / SLAM
- [COLMAP SfM (Schönberger 2016)](wiki/papers/schonberger2016_colmap-sfm.md) — canonical incremental SfM, CVPR 2016 · _2026-04-14_
- [GLOMAP](wiki/papers/pan2024_glomap.md) — global SfM matching COLMAP accuracy, orders of magnitude faster · _2026-04-14_
- [InstantSfM](wiki/papers/zhong2026_instantsfm.md) — GPU-native global SfM, 40x faster than COLMAP · _2026-04-12_
- [MP-SfM](wiki/papers/pataki2025_mp-sfm.md) — monocular depth/normal priors for robust incremental SfM · _2026-04-21_
- [VPGS-SLAM](wiki/papers/deng2026_vpgs-slam.md) — voxel-based progressive 3DGS SLAM for large scenes · _2026-04-12_
- [CuSfM](wiki/papers/yu2025_cusfm.md) — CUDA-accelerated SfM for autonomous driving · _2026-04-12_
- [MASt3R-SLAM](wiki/papers/murai2025_mast3r-slam.md) — real-time dense SLAM on MASt3R priors, 15 FPS · _2026-04-12_
- [MegaSaM](wiki/papers/li2025_megasam.md) — SfM from casual dynamic videos via motion segmentation · _2026-04-12_
- [DiffusionSfM](wiki/papers/zhao2025_diffusionsfm.md) — diffusion model for joint structure and motion prediction · _2026-04-12_
- [LoGeR](wiki/papers/zhang2025_loger.md) — long-context (19K frames) feedforward reconstruction via TTT · _2026-04-12_
- [ZipMap](wiki/papers/jin2026_zipmap.md) — linear-time bidirectional reconstruction, 20x faster than VGGT · _2026-04-12_
- [TTT3R](wiki/papers/chen2026_ttt3r.md) — training-free TTT patch to CUT3R, 2× pose improvement on long sequences · _2026-04-15_
- [DetectorFreeSfM (He 2023)](wiki/papers/he2023_detector-free-sfm.md) — coarse-to-fine detector-free SfM; quantized-match bridge + multi-view transformer refinement + track-topology adjustment; 340× less BA memory than PixSfM · _2026-04-21_
- [PixSfM (Lindenberger 2021)](wiki/papers/lindenberger2021_pixsfm.md) — feature-metric track refinement + BA; DetectorFreeSfM's direct baseline _(stub)_ · _2026-04-21_
- [FastVGGT (Shen 2025)](wiki/papers/shen2025_fastvggt.md) — training-free 4× VGGT speedup via three-part token merging; improves long-sequence pose via drift mitigation · _2026-04-21_
- [Faster-VGGT block-sparse (Wang 2025)](wiki/papers/wang2025_faster-vggt-block-sparse.md) — training-free 4× VGGT global-attention speedup via pooled-Q̄K̄ binary block mask; kernel-agnostic · _2026-04-21_
- [QuantVGGT (Feng 2025)](wiki/papers/feng2025_quantvggt.md) — first PTQ for VGGT, W4A4 at 98% accuracy retention; DSFQ + frame-aware calibration; CC-BY / Apache-2.0 · _2026-04-21_

### Mesh Reconstruction
- [PGSR](wiki/papers/chen2024_pgsr.md) — planar-constrained 3DGS with unbiased depth for high-fidelity meshes · _2026-04-15_
- [TSDF (Curless & Levoy 1996)](wiki/papers/curless1996_tsdf.md) — canonical volumetric fusion from range images · _2026-04-14_
- [SAM 3D](wiki/papers/chen2025_sam-3d.md) — single-image 3D reconstruction via latent flow matching (Meta) · _2026-04-14_
- [GeoSVR](wiki/papers/li2025_geosvr.md) — sparse voxels with uncertainty depth for SOTA surface reconstruction · _2026-04-12_
- [VA-GS](wiki/papers/li2025_va-gs.md) — multi-faceted view alignment for 3DGS surface quality · _2026-04-12_
- [AniSDF](wiki/papers/gao2025_anisdf.md) — fused-granularity SDF with anisotropic encoding for reflective objects · _2026-04-12_
- [SOF](wiki/papers/radl2025_sof.md) — sorted opacity fields, 3x faster optimization + 10x faster meshing · _2026-04-12_
- [MILo](wiki/papers/guedon2025_milo.md) — mesh-in-the-loop Gaussian splatting via Delaunay triangulation · _2026-04-12_
- [Confidence Mesh from 3DGS](wiki/papers/radl2026_confidence-mesh-3dgs.md) — per-Gaussian confidence for geometry/appearance balance · _2026-04-12_
- [VGG-T3](wiki/papers/elflein2026_vgg-t3.md) — O(n) feed-forward 3D reconstruction via test-time-trained MLPs · _2026-04-12_
- [SpatialLM](wiki/papers/mao2025_spatiallm.md) — LLM fine-tuned on point clouds for structured indoor modeling · _2026-04-12_

### Radiance Fields
- [ConceptFusion](wiki/papers/jatavallabhula2023_conceptfusion.md) — voxel-TSDF fusion of CLIP/DINO/AudioCLIP for open-set multimodal 3D mapping · _2026-04-15_
- [LangSVR](wiki/papers/wu2026_langsvr.md) — sparse-voxel representation with language + geometry dual distillation, one-stage · _2026-04-15_
- [Gaussian Grouping](wiki/papers/ye2024_gaussian-grouping.md) — per-Gaussian SAM-supervised identities for 3D segment + edit · _2026-04-15_
- [LangSplat](wiki/papers/qin2024_langsplat.md) — CLIP distilled into 3DGS with SAM-hierarchy supervision, 199× faster than LERF · _2026-04-15_
- [CLIP-GS](wiki/papers/jiao2025_clip-gs.md) — 3DGS-CLIP alignment for zero-shot 3D retrieval/classification · _2026-04-15_
- [Seg-Wild](wiki/papers/bao2025_seg-wild.md) — interactive 3DGS segmentation for in-the-wild photo collections · _2026-04-15_
- [GaussExplorer](wiki/papers/kim2026_gauss-explorer.md) — VLM + 3DGS for compositional embodied reasoning · _2026-04-15_
- [Mip-NeRF 360](wiki/papers/barron2022_mip-nerf-360.md) — unbounded NeRF via scene contraction + proposal network + distortion loss · _2026-04-15_
- [Zip-NeRF](wiki/papers/barron2023_zip-nerf.md) — anti-aliased hash-grid NeRF, 8× lower error than Mip-NeRF 360 · _2026-04-14_
- [VastGaussian](wiki/papers/lin2024_vastgaussian.md) — first real-time 3DGS for aerial/city scenes via partitioning + decoupled appearance · _2026-04-14_
- [GauSS-MI](wiki/papers/xie2025_gauss-mi.md) — real-time mutual information for active 3DGS view selection · _2026-04-12_
- [DroneSplat](wiki/papers/tang2025_dronesplat.md) — robust 3DGS from in-the-wild drone imagery · _2026-04-12_
- [CamP](wiki/papers/park2023_camp.md) — camera preconditioning for joint pose-NeRF optimization · _2026-04-12_
- [GS + Discretized SDF](wiki/papers/zhu2025_gs-discretized-sdf.md) — per-Gaussian SDF for relightable inverse rendering · _2026-04-12_
- [SVRaster](wiki/papers/sun2025_sparse-voxels-rasterization.md) — sparse voxel rasterization matching 3DGS quality, neural-free · _2026-04-12_
- [Multiview Geometric GS](wiki/papers/kim2025_multiview-geometric-gs.md) — MVS depth + normal regularization for SOTA explicit surfaces · _2026-04-12_
- [EA-3DGS](wiki/papers/guo2025_ea-3dgs.md) — efficient adaptive 3DGS with 5.2x compression for outdoor · _2026-04-12_

### MVS / Depth
- [COLMAP MVS (Schönberger 2016)](wiki/papers/schonberger2016_colmap-mvs.md) — pixelwise view selection PatchMatch MVS, canonical classical baseline · _2026-04-14_
- [MVSNet](wiki/papers/yao2018_mvsnet.md) — first end-to-end deep MVS via plane-sweep cost volume · _2026-04-14_
- [Multi-view Dense Matching](wiki/papers/chebbi2025_multiview-dense-matching.md) — similarity learning extended to multi-view without retraining · _2026-04-12_
- [Pow3R](wiki/papers/jang2025_pow3r.md) — conditioning DUSt3R with camera priors for native-resolution 3D · _2026-04-12_
- [MoGe (Wang 2025)](wiki/papers/wang2025_moge.md) — affine-invariant monocular geometry foundation model, CVPR 2025 _(stub)_ · _2026-04-21_

### Pose Estimation
- [Cameras as Rays](wiki/papers/zhang2024_cameras-as-rays.md) — Plucker ray-bundle pose estimation via diffusion · _2026-04-12_
- [MADPose (Yu 2025)](wiki/papers/yu2025_madpose.md) — affine-corrected depth-aware relative-pose solvers + hybrid LO-MSAC, CVPR 2025 Highlight · _2026-04-21_

### Feature Matching
- [MASt3R (Leroy 2024)](wiki/papers/leroy2024_mast3r.md) — 3D-grounded dense matching via dual-head DUSt3R + metric-scale pointmap loss + fast reciprocal-NN; +30 VCRE AUC over LoFTR+KBR on Map-free · _2026-04-21_
- [RoMa v2](wiki/papers/edstedt2025_roma-v2.md) — dense matcher with DINOv3 backbone, SOTA correspondence · _2026-04-12_
- [LoFTR (Sun 2021)](wiki/papers/sun2021_loftr.md) — detector-free semi-dense transformer matcher _(stub)_ · _2026-04-21_

### Fundamentals
- [CLIP](wiki/papers/radford2021_clip.md) — contrastive vision-language pretraining, zero-shot classification · _2026-04-14_
- [DINOv2](wiki/papers/oquab2023_dinov2.md) — self-supervised ViT foundation features on 142M curated images · _2026-04-14_
- [DINOv3](wiki/papers/simeoni2025_dinov3.md) — scaled DINOv2 with Gram anchoring + post-hoc text alignment · _2026-04-14_
- [Segment Anything (SAM)](wiki/papers/kirillov2023_sam.md) — promptable segmentation foundation model + SA-1B dataset · _2026-04-14_
- [SAM 3](wiki/papers/carion2026_sam-3.md) — Promptable Concept Segmentation (text + exemplars), 2× prior SOTA · _2026-04-14_
- [SD-RPN](wiki/papers/shi2026_self-distilled-roi.md) — self-distilled RoI predictors for fine-grained MLLM perception · _2026-04-12_
- [RADIOv2.5](wiki/papers/heinrich2025_radiov25.md) — agglomerative vision foundation model distilling CLIP+DINOv2+SAM · _2026-04-12_
- [Trident](wiki/papers/shi2024_open-vocab-segmentation.md) — training-free open vocabulary segmentation via CLIP+DINO+SAM · _2026-04-12_
- [Feed-Forward 3D Survey (Zhang 2025)](wiki/papers/zhang2025_feed-forward-3d-survey.md) — comprehensive taxonomy of feed-forward 3D methods, organized by output representation · _2026-04-12_
- [Feed-Forward 3D Survey (Wang 2026)](wiki/papers/wang2026_feed-forward-3d-scene-modeling.md) — second survey; problem-driven 5-axis taxonomy orthogonal to Zhang 2025's representation axis · _2026-04-21_

## Methods
- [3D Gaussian Splatting](wiki/methods/3d-gaussian-splatting.md) — explicit radiance field via anisotropic Gaussians · _updated 2026-04-14_
- [2D Gaussian Splatting](wiki/methods/2d-gaussian-splatting.md) — flat-Gaussian variant for better surfaces · _stub_
- [NeRF](wiki/methods/nerf.md) — neural radiance fields via MLP + volume rendering · _stub_
- [COLMAP](wiki/methods/colmap.md) — the standard SfM/MVS pipeline · _stub_
- [DUSt3R](wiki/methods/dust3r.md) — feed-forward two-view 3D reconstruction · _stub_
- [MASt3R](wiki/methods/mast3r.md) — DUSt3R + dense matching · _stub_
- [VGGT](wiki/methods/vggt.md) — Visual Geometry Grounded Transformer · _stub_
- [DROID-SLAM](wiki/methods/droid-slam.md) — dense visual SLAM with differentiable BA · _stub_
- [Marching Cubes](wiki/methods/marching-cubes.md) — isosurface extraction from volumetric fields · _stub_
- [GLOMAP](wiki/methods/glomap.md) — global SfM alternative to COLMAP · _stub_
- [DINOv2](wiki/methods/dinov2.md) — self-supervised ViT foundation features · _stub_
- [Zip-NeRF](wiki/methods/zip-nerf.md) — anti-aliased grid-based NeRF · _stub_
- [SAM](wiki/methods/sam.md) — promptable segmentation foundation model · _stub_
- [CLIP](wiki/methods/clip.md) — contrastive vision-language embedding · _stub_
- [Pow3R](wiki/methods/pow3r.md) — multi-conditioning pointmap predictor · _stub_
- [NeuS](wiki/methods/neus.md) — SDF via unbiased volume rendering · _stub_
- [Mip-NeRF 360](wiki/methods/mip-nerf-360.md) — unbounded-scene NeRF + 360° benchmark · _stub_
- [Metric3Dv2](wiki/methods/metric3dv2.md) — feed-forward metric mono-depth + normals · _stub_
- [Scaffold-GS](wiki/methods/scaffold-gs.md) — anchor-based 3DGS variant · _stub_
- [SGM](wiki/methods/sgm.md) — classical semi-global stereo matching · _stub_
- [CUT3R](wiki/methods/cut3r.md) — online/recurrent pointmap reconstruction · _stub_
- [CroCo](wiki/methods/croco.md) — cross-view completion pretraining · _stub_
- [RayDiffusion](wiki/methods/raydiffusion.md) — cameras-as-rays via diffusion · _stub_
- [LightGlue](wiki/methods/lightglue.md) — efficient sparse feature matcher · _stub_
- [DPT](wiki/methods/dpt.md) — Dense Prediction Transformer decoder · _stub_
- [VGGSfM](wiki/methods/vggsfm.md) — end-to-end differentiable SfM · _stub_
- [Depth Anything](wiki/methods/depthanything.md) — monocular depth foundation model · _stub_

## Concepts
- [Neural Implicit Surfaces](wiki/concepts/neural-implicit-surfaces.md) — SDF-based continuous surfaces; NeuS→Neuralangelo lineage · _stub_
- [Levenberg–Marquardt](wiki/concepts/levenberg-marquardt.md) — NLS optimizer behind bundle adjustment · _stub_
- [Foundation Model](wiki/concepts/foundation-model.md) — unified concept: CLIP, DINO, SAM, RADIO families + the frozen-backbone pattern · _2026-04-15_
- [Self-Supervised Learning](wiki/concepts/self-supervised-learning.md) — DINO→DINOv2→DINOv3 lineage, Gram anchoring, SSL vs. CLIP · _2026-04-15_
- [Segmentation (taxonomy)](wiki/concepts/segmentation.md) — semantic / instance / panoptic / promptable / open-vocabulary · _2026-04-15_
- [Open-Vocabulary Segmentation](wiki/concepts/open-vocabulary-segmentation.md) — Trident pattern and 3D lifting · _2026-04-15_
- [Structure from Motion](wiki/concepts/structure-from-motion.md) — classical pipeline and paradigms · _stub_
- [Multi-View Stereo](wiki/concepts/multi-view-stereo.md) — dense reconstruction from posed images · _stub_
- [TSDF](wiki/concepts/tsdf.md) — truncated signed distance field for depth fusion · _stub_
- [Bundle Adjustment](wiki/concepts/bundle-adjustment.md) — joint optimization of cameras + 3D points · _stub_
- [Signed Distance Field](wiki/concepts/signed-distance-field.md) — implicit surface representation · _stub_
- [Differentiable Rendering](wiki/concepts/differentiable-rendering.md) — gradient-based 3D optimization via rendering · _stub_
- [Test-Time Training](wiki/concepts/test-time-training.md) — adapting model weights at inference for long-context scaling · _stub_
- [Monocular Depth Estimation](wiki/concepts/monocular-depth-estimation.md) — single-image depth prediction · _stub_
- [Feature Matching](wiki/concepts/feature-matching.md) — correspondence finding between images · _stub_
- [Spherical Harmonics](wiki/concepts/spherical-harmonics.md) — basis functions for view-dependent color · _stub_
- [Vision Transformer](wiki/concepts/vision-transformer.md) — transformer architecture for visual tasks · _stub_
- [Feed-Forward 3D Problem Axes](wiki/concepts/feed-forward-problem-axes.md) — 5-axis problem-driven taxonomy for feed-forward 3D methods (Wang 2026) · _2026-04-21_

## Datasets
- [Tanks and Temples](wiki/datasets/tanks-and-temples.md) — laser-scanned large-scene reconstruction benchmark with F-score metric · _2026-04-15_

### Dataset papers
- [Tanks and Temples (Knapitsch 2017)](wiki/papers/knapitsch2017_tanks-and-temples.md) — the benchmark paper · _2026-04-15_

## People
_(empty)_

## Threads
- [Lifting Foundation Models to 3D](wiki/threads/lifting-foundation-models-to-3d.md) — SAM/CLIP/DINO distilled into 3DGS/voxel primitives, 3 OPs · _updated 2026-04-18_
- [Foundation Features for Geometry](wiki/threads/foundation-features-for-geometry.md) — frozen DINO backbones replacing SIFT across SfM / matching / depth · _updated 2026-04-18_
- [Open-Vocab 2D Composition](wiki/threads/open-vocab-2d-composition.md) — CLIP + DINO + SAM as complementary backbones (Trident, SAM 3, RADIO), 3 OPs · _updated 2026-04-18_
- [Radiance Field Evolution](wiki/threads/radiance-field-evolution.md) — NeRF → 3DGS lineage, 3 OPs (quality-per-scene / city-scale / neural-free) · _updated 2026-04-21_
- [Gaussian-to-Mesh Pipelines](wiki/threads/gaussian-to-mesh-pipelines.md) — extracting meshes from Gaussian splats, 3 OPs (regularized-3dgs / mesh-in-loop / natively-extractable) · _updated 2026-04-18_
- [Feed-Forward Structure from Motion](wiki/threads/feed-forward-structure-from-motion.md) — three-tier SfM taxonomy: classical → hybrid → fully feed-forward · _updated 2026-04-21_
- [GPU-Native SfM (Tier 1)](wiki/threads/gpu-native-sfm.md) — 2 OPs (general-purpose InstantSfM / sequential-slam-prior CuSfM) · _updated 2026-04-18_
- [Monocular Depth Estimation](wiki/threads/mono-depth-estimation.md) — mono depth as load-bearing prior for SfM/3DGS/MVS · _updated 2026-04-18_
- [Nerfstudio + gsplat Codebase](wiki/threads/nerfstudio.md) — local visiofacto fork architecture reference · _updated 2026-04-18_
- [VLM Reasoning over 3D Scenes](wiki/threads/vlm-reasoning-over-3d-scenes.md) — inference-time VLM reasoning on reconstructed 3DGS scenes (GaussExplorer) · _2026-04-18_
- [Generative 3D from 2D Priors](wiki/threads/generative-3d-from-2d-priors.md) — single-image generative 3D (SAM 3D, latent flow-matching) · _2026-04-21_
- [LLM-Native Structured Scenes](wiki/threads/llm-native-structured-scenes.md) — point-cloud → LLM-emitted structured scene scripts (SpatialLM) · _2026-04-18_
- [Relative Pose Estimation](wiki/threads/relative-pose-estimation.md) — depth-aware two-view pose (MADPose); calibrated / shared-focal / uncalibrated · _updated 2026-04-21_

## Designs
- [Language-Grounded 3DGS 2026 — Research-Synthesis Design](wiki/designs/language-grounded-3dgs-2026.md) — realizes Bet #020; composes SAM 3 + RADIOv2.5 + DINOv3 + LangSplat-style autoencoder on a 3DGS substrate · _updated 2026-04-18_
- [Language-Grounded 3DGS — Nerfstudio / Visiofacto Implementation Plan](wiki/designs/language-grounded-3dgs-nerfstudio.md) — 9-phase build plan for Bet #020; `visiofacto-lang` method · _updated 2026-04-18_
- [CoMe Integration into Nerfstudio/Visiofacto](wiki/designs/come-integration-nerfstudio.md) — direct CoMe re-implementation (no bet); confidence + variance losses + appearance module · _updated 2026-04-18_
- [Re-ingest tracker](wiki/designs/reingest-tracker.md) — workflow tracker (meta; not a pipeline design) · _updated 2026-04-18_

## Ideas

91 idea pages live in [wiki/ideas/](wiki/ideas/). Listing them individually in this index would push it past the 300-line split threshold; browse the directory directly or query by stage via `lint stage-coverage`. Key ideas referenced by Bets #001–#030: CoMe confidence, Gaussian Grouping identity, SAM 3 concept segmentation, LangSplat autoencoder, InstantSfM depth-constrained Jacobian, VastGaussian partitioning + decoupled appearance, DINOv3 Gram anchoring, RADIOv2.5 agglomerative distillation, MILo mesh-in-loop, TTT3R closed-form LR, MP-SfM uncertainty-calibration + bilateral-normal-integration + matcher-score next-view, DetectorFreeSfM bridge + transformer refinement + track-topology adjustment, MADPose affine-corrected solvers + depth-induced scoring + hybrid LO-MSAC, MASt3R dual-head matching + metric-scale pointmap loss + fast reciprocal-NN + coarse-to-fine window covering, **VGGT-compression family** (FastVGGT three-part token merge + block-sparse pooled-Q̄K̄ attention + QuantVGGT DSFQ/NFDS), plus 76 more.

## Stages

96 stage pages live in [wiki/stages/](wiki/stages/) — typed slots for pipeline composition. Organized by domain: `radiance-fields.*` (16), `sfm.*` (17), `feed-forward-sfm.*` (18), `pose-estimation.*` (3), `open-vocab-2d.*` (5), `lifting-foundation-models.*` (12), `mvs.*` (4), `mesh-reconstruction.*` (2), `feature-matching.*` (3), `foundation-features.*` (1), `vlm-reasoning.*` (3), `generative-3d.*` (3), `llm-structured-scenes.*` (3), plus `relighting.*`, `active-reconstruction.*`. New 2026-04-21: `sfm.feature-track-refinement`, `sfm.track-topology-adjustment`, `pose-estimation.relative-pose-solver`, `pose-estimation.robust-estimator-scoring`, `pose-estimation.hybrid-robust-estimator`, `feature-matching.reciprocal-matching`, `feed-forward-sfm.token-compaction`, `feed-forward-sfm.numerical-precision`, `feed-forward-sfm.ptq-calibration-sampling`. Browse the directory or query via `lint stage-coverage`.

## Meta
- [License Audit](wiki/meta/license-audit.md) — commercial-use readiness of all 55 wiki papers; bet-level commercial-readiness table; remediation checklist · _2026-04-18_

---

_Last rebuilt: 2026-04-21 · 65 papers, 14 methods, 16 concepts, 13 threads, 4 designs, 91 ideas, 96 stages, 1 meta_
