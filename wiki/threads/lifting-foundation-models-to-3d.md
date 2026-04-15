---
title: Lifting Foundation Models to 3D
type: thread
tags: [3dgs, sam, clip, dino, segmentation, open-vocabulary, scene-editing]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/ye2024_gaussian-grouping.md, wiki/papers/qin2024_langsplat.md, wiki/papers/jiao2025_clip-gs.md, wiki/papers/bao2025_seg-wild.md, wiki/papers/kim2026_gauss-explorer.md, wiki/papers/chen2025_sam-3d.md, wiki/papers/carion2026_sam-3.md, wiki/papers/jatavallabhula2023_conceptfusion.md, wiki/papers/wu2026_langsvr.md]
status: draft
---

## Working hypothesis

The 2024–2026 literature converges on a consistent pattern: take a 2D foundation model ([[sam|SAM]], [[clip|CLIP]], [[dinov2|DINO]]) and **lift its output into a 3D representation — usually per-primitive features on [[3d-gaussian-splatting|3DGS]]** — so that downstream 3D tasks (segment, edit, query, reason) can operate natively in 3D. The granularity of the lifted signal (identity label, feature vector, language embedding) and the training recipe (per-scene distillation vs. generalizable alignment) vary, but the core pattern is stable enough to call a paradigm.

## Two axes

**Axis 1 — what is lifted:**
- **Identity / mask membership** (low-dim, SAM-sourced)
- **Language features** (high-dim, CLIP-sourced, often compressed via autoencoder)
- **Composite / multi-channel** (identity + appearance + lighting-invariance)

**Axis 2 — how it generalizes:**
- **Per-scene distillation** — train a language/identity field on each new capture. Best quality per scene, no cross-scene transfer.
- **Scene-level alignment** — align a scene encoder with CLIP once; inference runs on unseen scenes zero-shot.
- **Inference-time VLM reasoning** — don't lift features at all; render views and hand them to a VLM.

## Evidence

### SAM-lifted identities (per-Gaussian masks)
- [[ye2024_gaussian-grouping|Gaussian Grouping]] (Ye 2024, ECCV) — canonical recipe. Per-Gaussian **Identity Encoding** supervised by SAM masks with cross-view association + 3D spatial-consistency regularization. Enables object removal/inpainting/recomposition.
- [[bao2025_seg-wild|Seg-Wild]] (Bao 2025) — extends to in-the-wild photo collections with transient occluders. Adds multi-dimensional embeddings + the Spiky 3D Gaussian Cutter (projects Gaussians onto SAM masks, cuts those that cross boundaries).

### CLIP-lifted language fields
- [[qin2024_langsplat|LangSplat]] (Qin 2024, CVPR) — distills CLIP into per-Gaussian latents via a scene-specific autoencoder; uses SAM's mask hierarchy to replace LERF's expensive multi-scale queries. **199× faster than LERF** at 1440×1080. Canonical 3D-OVS method.
- [[jiao2025_clip-gs|CLIP-GS]] (Jiao 2025, ICCV) — different philosophy: contrastively align a **scene-level** 3DGS embedding with CLIP image+text via a GS Tokenizer. Generalizes across scenes zero-shot; outperforms point-cloud multimodal models (ULIP, Uni3D, OpenShape) on retrieval and classification.

### Generative single-image 3D under the SAM umbrella
- [[chen2025_sam-3d|SAM 3D]] (Chen et al., Meta 2025) — single-image → per-object geometry + texture + scene layout via two-stage latent flow-matching. Demonstrates that lifted foundation models can *generate* 3D from a single image, not just encode reconstructions.

### Voxel-based lifting (the non-Gaussian lane)
- [[jatavallabhula2023_conceptfusion|ConceptFusion]] (Jatavallabhula 2023, RSS) — fuses CLIP + DINO + AudioCLIP features into a **voxel-hashed [[tsdf|TSDF]] map** via classical online integration. Supports text / image / audio / click queries on one shared embedding space. Robotics-oriented; online; no novel-view synthesis.
- [[wu2026_langsvr|LangSVR]] (Wu 2026) — sparse-voxel rasterization (on [[sun2025_sparse-voxels-rasterization|SVRaster]]) with four co-trained fields (appearance, density, feature, confidence), distilling both CLIP *and* a monocular-depth geometry foundation model in a one-stage pipeline. Outperforms LangSplat / Gaussian Grouping / SVRaster on combined mIoU + PSNR.

These show the lifting pattern is **representation-agnostic**: the same "2D foundation features → per-primitive attribute" template works whether the primitive is a Gaussian, a sparse voxel, or a TSDF cell.

### Inference-time VLM reasoning on top of 3DGS
- [[kim2026_gauss-explorer|GaussExplorer]] (Kim 2026) — skips per-Gaussian feature lifting entirely. Retrieves + synthesizes novel views from 3DGS and hands them to a pretrained VLM for compositional reasoning. Represents the "consumer" side of lifted 3D: how downstream agents use it.

### 2D upstream that feeds this thread
- [[carion2026_sam-3|SAM 3]] (2026) — Promptable Concept Segmentation with text + exemplars. The natural next 2D source to lift; no 3D-lifted follow-up is in the wiki yet.
- [[shi2024_open-vocab-segmentation|Trident]] (2024) — 2D composition of CLIP + DINO + SAM; the 2D baseline the 3D lifting methods inherit conceptually.

## Patterns and tradeoffs

| Lift style | Strength | Weakness |
|---|---|---|
| SAM identity only | Clean segmentation + editing | No text alignment |
| CLIP features distilled per-scene (LangSplat) | High per-scene quality, OVS-ready | No cross-scene generalization; per-scene training |
| Scene-level CLIP alignment (CLIP-GS) | Zero-shot across scenes | Lower per-scene precision; requires pretrained triplet dataset |
| Inference-time VLM (GaussExplorer) | Zero training on the 3D side | Slow inference; VLM hallucinations |
| Voxel-TSDF fusion (ConceptFusion) | Online, multimodal, robotics-ready | No rendering; voxel resolution trades off |
| Sparse voxel + dual distillation (LangSVR) | One-stage, adds geometric grounding | Technical report; geometry-FM dependency |

## Open questions
- **Memory**: per-Gaussian features at scene scale explode memory. LangSplat's autoencoder is one answer; quantization / hashing could help.
- **Lifting granularity**: per-Gaussian vs. per-cluster vs. per-mask — no principled theory for which wins at which task. Empirical so far.
- **Densification & pruning interaction**: lifted features must be inherited/split/merged alongside geometry. All current methods handle this ad hoc.
- **3D-native foundation models**: will the pattern invert? Rather than lift 2D features up, train backbones directly on 3DGS / volumetric data (CLIP-GS is a first step; OpenShape, Uni3D precede).
- **Benchmarks**: 3D OVS and 3D instance-segmentation benchmarks are nascent; cross-paper comparison is currently noisy.

## Related threads
- [[foundation-features-for-geometry]] — same foundation models, but used for *geometric* regression (matching, depth, pose) rather than segmentation/language.
- [[radiance-field-evolution]] — the underlying 3DGS/NeRF representation story.
- [[gaussian-to-mesh-pipelines]] — complementary 3DGS direction (surfaces) that has not yet seriously intersected with foundation-feature lifting.
- [[open-vocabulary-segmentation]] — the 2D task whose lift into 3D this thread tracks.
