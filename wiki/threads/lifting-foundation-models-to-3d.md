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

---

## Goal & success criteria

Produce a 3D representation (Gaussian, voxel, TSDF, or implicit) whose primitives carry foundation-model features sufficient to support open-vocabulary queries (text/image/audio/click), instance segmentation, editing, and VLM reasoning. "Better" = higher 3D-OVS mIoU / instance IoU, lower per-scene training cost, and modularity (new 2D foundation model → drop-in upgrade, no retrain of the 3D pipeline).

## Current SOTA pipelines (as of 2026-04-15)

**Pipeline I — Per-scene CLIP distillation onto 3DGS** (default for NVS-first applications):
1. 3DGS primitives trained per scene.
2. CLIP embeddings extracted per view via SAM mask hierarchy. Paper: [qin2024_langsplat].
3. Scene-specific autoencoder compresses 512-D CLIP → low-D latent per Gaussian.
4. Rendering + query by cosine similarity.

**Pipeline II — SAM-mask-lifted per-Gaussian identities** (default for editing):
1. 3DGS primitives.
2. Per-Gaussian Identity Encoding supervised by SAM masks with cross-view association + 3D spatial-consistency. Paper: [ye2024_gaussian-grouping].
3. For in-the-wild captures: multi-dim embeddings + Spiky 3D Gaussian Cutter + transient appearance embedding. Paper: [bao2025_seg-wild].

**Pipeline III — Scene-level contrastive alignment** (default for zero-shot cross-scene):
1. 3DGS pre-computed.
2. GS Tokenizer → transformer init from point-cloud FM → contrastive with CLIP image+text. Paper: [jiao2025_clip-gs].

**Pipeline IV — Voxel TSDF + multi-modal feature fusion** (default for robotics/online):
1. RGB-D stream → voxel-hashed TSDF.
2. Per-frame pixel features from CLIP + DINO + AudioCLIP → per-voxel running weighted average. Paper: [jatavallabhula2023_conceptfusion].
3. Multimodal query by cosine on shared embedding space.

**Pipeline V — Sparse-voxel one-stage unified (language + geometry distillation)**:
1. SVRaster primitives. Paper: [sun2025_sparse-voxels-rasterization].
2. Four co-trained fields: appearance + density + feature + confidence. Paper: [wu2026_langsvr].
3. Dual distillation: CLIP features + monocular-depth geometry FM.

**Pipeline VI — Inference-time VLM reasoning** (default for compositional queries):
1. 3DGS scene + query → view retrieval + novel-view synthesis.
2. Composite views → pretrained VLM. Paper: [kim2026_gauss-explorer].

**Pipeline VII — Single-image generative 3D**:
1. Single RGB image → latent flow-matching shape gen + layout. Paper: [chen2025_sam-3d].

**Pipeline VIII — LLM-native structured indoor understanding**:
1. Point cloud + Sonata encoder → LLM generating Python-like scene scripts. Paper: [mao2025_spatiallm].

## Pipeline lineage

- 2023 · voxel TSDF lifting of CLIP + DINO + AudioCLIP. Driver: [jatavallabhula2023_conceptfusion].
- 2024 · NeRF-based CLIP field (LERF) → 3DGS CLIP autoencoder distillation. Driver: [qin2024_langsplat].
- 2024 · 3DGS identity encoding from SAM. Driver: [ye2024_gaussian-grouping].
- 2025 · in-the-wild identity segmentation. Driver: [bao2025_seg-wild].
- 2025 · contrastive scene-level alignment (3DGS ↔ CLIP). Driver: [jiao2025_clip-gs].
- 2025 · LLM-native structured indoor modeling. Driver: [mao2025_spatiallm].
- 2025 · single-image generative scene 3D. Driver: [chen2025_sam-3d].
- 2026 · one-stage sparse-voxel unified (language + geometry). Driver: [wu2026_langsvr].
- 2026 · VLM-reasoning-over-3DGS. Driver: [kim2026_gauss-explorer].

## Candidate components / not yet integrated

- **SAM 3 masks with native instance IDs** ([carion2026_sam-3]) as Gaussian Grouping's supervision — kills the cross-view association module; mechanism is "drop in SAM 3, delete association loop." Highest-priority synthesis bet in the thread.
- **RADIOv2.5 as the per-Gaussian feature source** ([heinrich2025_radiov25]) instead of raw CLIP — one backbone distillation replaces CLIP + DINO + SAM leg-by-leg.
- **DINOv3 spatial coherence** for the LangSplat autoencoder's regularization slot (LangSplat explicitly dropped DINO; re-adding DINOv3 might revert that choice).
- **TTT3R-style per-token update** applied to LangSplat's autoencoder training — per-embedding learning rate from render-confidence.

## Open questions & synthesis bets

- **Synthesis bet 1**: *Gaussian Grouping with SAM 3 supervision* (identity IDs pre-aligned across views → delete association module). Mixes [ye2024_gaussian-grouping] + [carion2026_sam-3]. Expected: 5–10% mIoU gain + simpler training.
- **Synthesis bet 2**: *LangSplat with RADIOv2.5 as the feature source* + *CoMe confidence as the per-Gaussian feature weighting*. Mixes [qin2024_langsplat] + [heinrich2025_radiov25] + [radl2026_confidence-mesh-3dgs]. Unified feature encoder + unified uncertainty.
- **Synthesis bet 3**: *Wu 2026 LangSVR's one-stage recipe with SAM 3 as the mask source and DINOv3 as the geometry prior*. Mixes [wu2026_langsvr] + [carion2026_sam-3] + [simeoni2025_dinov3]. Every component upgraded to the 2026 state.
- **Synthesis bet 4**: *ConceptFusion-style online voxel fusion with RADIOv2.5 features + VPGS-SLAM's progressive submaps*. Mixes [jatavallabhula2023_conceptfusion] + [heinrich2025_radiov25] + [deng2026_vpgs-slam]. Online, multimodal, city-scale lifted 3D — no paper does this.
- **Memory**: per-Gaussian features at scene scale explode memory. Open: distilled / quantized per-primitive features. EA-3DGS-style codebook VQ applied to feature channels? No paper tries this.
- **3D-native foundation models**: CLIP-GS trains on 3DGS directly. Open question: will the field invert — skip CLIP entirely, pretrain FMs on 3D?
- **Benchmarks**: 3D-OVS and 3D instance-segmentation benchmarks remain noisy; cross-paper comparison frequently apples-to-oranges.

## Contradictions & tensions

- **Per-scene vs. generalizable**: LangSplat (per-scene autoencoder) wins on per-scene quality; CLIP-GS (scene-level contrastive) wins on zero-shot transfer. No paper unifies both on the same benchmarks — the comparison is always conditioned on the scenario.
- **Representation substrate**: 3DGS (LangSplat, Gaussian Grouping) vs. sparse voxel (LangSVR, ConceptFusion voxel-TSDF) — both claim the lifting pattern, but the substrate choice shapes what downstream tasks work (NVS + editing → 3DGS; online robotics → voxel).
