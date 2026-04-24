---
title: Lifting Foundation Models to 3D
type: thread
tags: [3dgs, sam, clip, dino, segmentation, open-vocabulary, scene-editing]
created: 2026-04-15
updated: 2026-04-24
sources: [wiki/papers/ye2024_gaussian-grouping.md, wiki/papers/qin2024_langsplat.md, wiki/papers/jiao2025_clip-gs.md, wiki/papers/bao2025_seg-wild.md, wiki/papers/kim2026_gauss-explorer.md, wiki/papers/chen2025_sam-3d.md, wiki/papers/carion2026_sam-3.md, wiki/papers/jatavallabhula2023_conceptfusion.md, wiki/papers/wu2026_langsvr.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/ranzinger2026_c-radiov4.md, wiki/papers/meng2026_seen2scene.md, wiki/designs/language-grounded-3dgs-2026.md]
operating_points: [op:per-scene-3dgs, op:zero-shot-3dgs, op:voxel-online]
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

## Goal

Produce a 3D representation (Gaussian, voxel, TSDF, or implicit) whose primitives carry foundation-model features sufficient to support open-vocabulary queries (text/image/audio/click), instance segmentation, editing, and VLM reasoning. Better = higher 3D-OVS mIoU / instance IoU, lower per-scene training cost, and modularity (new 2D foundation model → drop-in upgrade, no retrain of the 3D pipeline).

## Goal contract (optional, structured)

```yaml
metric: [3d-OVS-mIoU, instance-IoU, per-scene-train-min, query-latency-ms]
target_regime: [posed-images | RGB-D stream, bounded-scene | room | city, static-or-dynamic]
constraints: [2d-backbone-upgradeable-without-3d-retrain, commercial-license-preferred]
required_capabilities: [open-vocab-query, instance-segmentation, scene-editing, foundation-feature-lifting]
```

## SOTA pipelines

**Migration note (2026-04-18).** The pre-migration thread listed 8 parallel pipelines (I–VIII). The §6.9 Pareto cap limits threads to 3 OPs. This migration consolidated the 3DGS and voxel-lifting pipelines (I, II, III, IV, V) into 3 OPs below. Pipelines VI–VIII split into sibling threads the same day; see "Sibling threads (split 2026-04-18)" below.

### op:per-scene-3dgs (default for NVS-first + editing; consolidates old Pipelines I + II)

1. 3DGS primitives trained per scene.
2. Per-Gaussian feature source — choose one:
   - CLIP embeddings extracted per view via SAM mask hierarchy, compressed via scene-specific autoencoder (→ low-D latent per Gaussian). Paper: [[qin2024_langsplat]] (LangSplat).
   - SAM-mask-supervised Identity Encoding with cross-view association + 3D spatial-consistency regularization. Paper: [[ye2024_gaussian-grouping]]. In-the-wild extension: multi-dim embeddings + Spiky 3D Gaussian Cutter + transient appearance. Paper: [[bao2025_seg-wild]].
3. Rendering + query by cosine similarity (for CLIP-latent variant) or instance-ID match (for SAM-identity variant).

### op:zero-shot-3dgs (default when transfer across scenes matters; old Pipeline III)

1. 3DGS pre-computed.
2. GS Tokenizer → transformer init from point-cloud FM → contrastive with CLIP image+text. Paper: [[jiao2025_clip-gs]].
3. Zero-shot query on unseen scenes via shared embedding space.

### op:voxel-online (default for robotics / online / multimodal; consolidates old Pipelines IV + V)

1. Primitive: voxel-hashed TSDF (online RGB-D) or SVRaster primitives (batched).
2. Per-voxel feature fusion — choose one:
   - Per-frame pixel features from CLIP + DINO + AudioCLIP → per-voxel running weighted average. Paper: [[jatavallabhula2023_conceptfusion]] (ConceptFusion). Online, multimodal.
   - Four co-trained fields (appearance + density + feature + confidence) with dual distillation (CLIP + monocular-depth geometry FM). Paper: [[wu2026_langsvr]] (LangSVR).
3. Multimodal query by cosine on shared embedding space.

## Pipeline lineage

- 2023 · op:voxel-online · voxel TSDF lifting of CLIP + DINO + AudioCLIP. Driver: [[jatavallabhula2023_conceptfusion]].
- 2024 · op:per-scene-3dgs · NeRF-based CLIP field (LERF) → 3DGS CLIP autoencoder distillation. Driver: [[qin2024_langsplat]].
- 2024 · op:per-scene-3dgs · 3DGS identity encoding from SAM. Driver: [[ye2024_gaussian-grouping]].
- 2025 · op:per-scene-3dgs · in-the-wild identity segmentation. Driver: [[bao2025_seg-wild]].
- 2025 · op:zero-shot-3dgs · new OP: contrastive scene-level alignment (3DGS ↔ CLIP). Driver: [[jiao2025_clip-gs]].
- 2026 · op:voxel-online · one-stage sparse-voxel unified (language + geometry). Driver: [[wu2026_langsvr]].
- 2026-04-18 · thread-split · Pipelines VI–VIII removed to sibling threads [[vlm-reasoning-over-3d-scenes]], [[generative-3d-from-2d-priors]], [[llm-native-structured-scenes]].
- 2026-04-22 · op:per-scene-3dgs, op:voxel-online · candidate feature-source bump: [[radiov25-agglomerative-distillation_heinrich2025|RADIOv2.5]] → [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] · driver: [[ranzinger2026_c-radiov4]] · rationale: commercial license (NVIDIA Open Model License) unblocks production use of RADIO-based lifting; 2026 teacher set (SigLIP2/DINOv3/SAM3); cleaner feature boundaries plausibly improve per-primitive lifting. Not yet a SOTA swap — remains a candidate pending LangSplat-style ablation on LERF / ScanNet++.

## Candidate components / not yet integrated

- **SAM 3 masks with native instance IDs** ([[carion2026_sam-3]]) as Gaussian Grouping's supervision — kills the cross-view association module; mechanism is "drop in SAM 3, delete association loop." Target OP: `op:per-scene-3dgs`. Highest-priority bet in the thread.
- **[[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] as the per-Gaussian feature source** (preferred as of 2026-04-22; supersedes the prior RADIOv2.5 candidate which remains listed as fallback under NSCL research-only license). Paper: [[ranzinger2026_c-radiov4]]. One backbone distillation replaces CLIP + DINO + SAM leg-by-leg, under the NVIDIA Open Model License (commercial use permitted). Cleaner per-image feature boundaries (Fig 1 of the paper) plausibly translate to cleaner per-Gaussian lifts. Target OP: `op:per-scene-3dgs`, `op:voxel-online`.
- **[[radiov25-agglomerative-distillation_heinrich2025|RADIOv2.5]] as a fallback feature source** (research-only NSCL) — kept in candidates for direct comparability with published RADIOv2.5-based baselines.
- **DINOv3 spatial coherence** for the LangSplat autoencoder's regularization slot (LangSplat explicitly dropped DINO; re-adding DINOv3 might revert that choice). Target OP: `op:per-scene-3dgs`.
- **TTT3R-style per-token update** applied to LangSplat's autoencoder training — per-embedding learning rate from render-confidence. Target OP: `op:per-scene-3dgs`.
- **Pipeline IX proposal — feature-augmented 3DGS with 2026-stack lifting** ([[language-grounded-3dgs-2026]]) — full pipeline composing SAM 3 native track-IDs + RADIOv2.5 unified features + DINOv3 depth/normal priors + LangSplat-style per-scene autoencoder into a one-stage 3DGS training recipe, plus two contributions no single paper provides: (a) a densification/pruning invariant rule set (NULL_ID sentinel, entropy-gated splits, boundary-Gaussian protection, no-reset on opacity-reset); (b) a precomputed interaction layer (VQ codebook + CSR identity hash + per-instance mean latents + FAISS IVF-PQ fallback) targeting click < 5 ms and text < 100 ms at 1 M Gaussians. **Kept in candidates** — not promoted to SOTA — until Build-Sequence verification passes (see design §Verification Plan). Target OP: `op:per-scene-3dgs`. Nerfstudio / visiofacto implementation plan: [[language-grounded-3dgs-nerfstudio]].

### Sibling threads (split 2026-04-18)

The 2026-04-18 migration split three structurally distinct task classes out of this thread into their own sibling threads. This thread tracks per-scene lifted-feature 3DGS/voxel methods; the sibling threads track tasks where the primary output is not a feature-augmented radiance field.

- [[vlm-reasoning-over-3d-scenes]] — inference-time VLM reasoning on reconstructed 3D scenes ([[kim2026_gauss-explorer]] / Pipeline VI origin).
- [[generative-3d-from-2d-priors]] — single-image-to-3D generative scene synthesis ([[chen2025_sam-3d]] / Pipeline VII origin).
- [[llm-native-structured-scenes]] — point-cloud → LLM-emitted structured scene scripts ([[mao2025_spatiallm]] / Pipeline VIII origin).

Cross-thread synthesis bets connecting this thread to the siblings live in the sibling threads' own "Open questions & synthesis bets" sections (Bets #021, #023, #025).

## Open questions & synthesis bets

### Bet #013 — Gaussian Grouping with SAM 3 native instance IDs
status: proposed
combines: [[ye2024_gaussian-grouping]], [[carion2026_sam-3]]
stage_target: lifting-foundation-models.identity-supervision
op_target: op:per-scene-3dgs
confidence: high
magnitude: substantial
cost: days
breakage_risk: low
hypothesis: SAM 3's native cross-view instance IDs obsolete Gaussian Grouping's cross-view association module. Drop in SAM 3, delete the association loss — simpler training, better IDs.
expected_gain: 5–10% mIoU gain + ~20–30% training-time reduction (no association loss to converge).
risk: SAM 3 IDs may be unstable under large viewpoint change; some fallback association may still be needed.
validating_experiment: Replace Gaussian Grouping's mask supervision with SAM 3 outputs; ablate with/without the original association loss on LERF benchmarks.
triggers: [ingest-of-idea:sam3-id-stability-analysis]
created: 2026-04-15 · updated: 2026-04-18

### Bet #014 — LangSplat with C-RADIOv4 features + CoMe per-Gaussian weighting
status: proposed
combines: [[qin2024_langsplat]], [[cradiov4-agglomerative-distillation_ranzinger2026]], [[radl2026_confidence-mesh-3dgs]]
stage_target: lifting-foundation-models.feature-source
op_target: op:per-scene-3dgs
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: Replace CLIP as LangSplat's feature source with [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] (unified backbone, subsumes SigLIP2+DINOv3+SAM3) and weight per-Gaussian feature updates by CoMe's self-supervised confidence. Unified feature encoder + unified uncertainty, under a commercial license.
expected_gain: 3D-OVS mIoU gain over LangSplat baseline; cleaner boundaries from CoMe confidence masking out low-confidence Gaussians.
risk: C-RADIOv4's feature dimension differs from CLIP; autoencoder must be re-tuned. CoMe confidence was designed for geometry, may not map well to semantic features.
validating_experiment: LangSplat with {CLIP, C-RADIOv4, C-RADIOv4+CoMe-weighting} on LERF 3D-OVS + ScanNet++.
triggers: [ingest-of-idea:confidence-weighted-semantic-feature-lifting]
**Backbone bump 2026-04-22**: was RADIOv2.5 (NSCL, non-commercial). Upgraded to C-RADIOv4 for commercial licensability + better feature quality. Published RADIOv2.5-LangSplat ablations remain the direct comparison baseline.
created: 2026-04-15 · updated: 2026-04-22

### Bet #015 — LangSVR with SAM 3 masks + DINOv3 geometry prior
status: proposed
combines: [[wu2026_langsvr]], [[carion2026_sam-3]], [[simeoni2025_dinov3]]
stage_target: lifting-foundation-models.multi-field-training
op_target: op:voxel-online
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: Wu 2026's one-stage SVRaster + 4-field recipe already distills CLIP + mono-depth. Upgrading every component to the 2026 state — SAM 3 as mask source (was: SAM), DINOv3 as geometry prior (was: unnamed depth FM) — should close the remaining gap vs. Pipeline B (SAM 3) on segmentation + Pipeline I (LangSplat) on OVS.
expected_gain: ≥3% mIoU + ≥0.3 PSNR over LangSVR baseline on joint benchmarks.
risk: Already a technical report; architecture may be fragile to component swaps. Multi-teacher distillation may stall.
validating_experiment: Component-swap ablation {LangSVR, +SAM3, +DINOv3, +both} on ScanNet++ segmentation + LERF OVS.
triggers: [ingest-of-idea:multi-teacher-voxel-distillation]
created: 2026-04-15 · updated: 2026-04-18

### Bet #016 — ConceptFusion + C-RADIOv4 + VPGS-SLAM progressive submaps
status: proposed
combines: [[jatavallabhula2023_conceptfusion]], [[cradiov4-agglomerative-distillation_ranzinger2026]], [[deng2026_vpgs-slam]]
stage_target: lifting-foundation-models.online-fusion
op_target: op:voxel-online
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: ConceptFusion is RGB-D → voxel-TSDF + multimodal features (2023); VPGS-SLAM gives progressive submaps + loop closure (2026); C-RADIOv4 gives unified per-frame features under commercial license (2026), with ViTDet mode available for online latency. Composing the three yields an online, multimodal, city-scale lifted 3D representation — no paper does this.
expected_gain: First demonstration of km²-scale online lifted 3D; qualitative validation on extended indoor+outdoor sequences.
risk: Loop-closure re-voxelization invalidates accumulated features; no paper handles this. Even with ViTDet mode, C-RADIOv4 inference cost may dominate the online budget — measure before committing.
validating_experiment: Build the three-component pipeline; evaluate on extended ScanNet++ sequences + a custom outdoor walk. Benchmark C-RADIOv4-SO400M-VDT12 (smallest + fastest) as the per-frame feature source.
triggers: [ingest-of-idea:feature-preserving-loop-closure]
**Backbone bump 2026-04-22**: was RADIOv2.5. Upgraded to C-RADIOv4 for commercial licensability and ViTDet deployment mode (critical for online robotics budget).
created: 2026-04-15 · updated: 2026-04-22

### Bet #020 — Feature-augmented 3DGS with the 2026-stack (Pipeline IX)
status: in-design
combines: [[carion2026_sam-3]], [[cradiov4-agglomerative-distillation_ranzinger2026]], [[simeoni2025_dinov3]], [[qin2024_langsplat]], [[ye2024_gaussian-grouping]]
stage_target: lifting-foundation-models.per-primitive-feature-lifting
op_target: op:per-scene-3dgs
confidence: med
magnitude: substantial
cost: months
breakage_risk: med
hypothesis: Re-compose the 2024-era lifting baselines (LangSplat, Gaussian Grouping, Trident composition) on the 2026 foundation stack (SAM 3 native IDs, [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] unified features, DINOv3 geometric priors), plus two gaps no prior paper covers: (a) densification/pruning invariants (NULL_ID sentinel, entropy-gated splits, boundary-Gaussian protection, no-reset on opacity-reset); (b) precomputed interaction layer (VQ codebook + CSR identity hash + per-instance mean latents + FAISS IVF-PQ fallback) targeting click <5 ms, text <100 ms at 1 M Gaussians.
expected_gain: LangSplat-quality OVS + Gaussian-Grouping-quality identity + interaction latency at deployment-ready scale. No single paper delivers all three simultaneously.
risk: Densification invariants are the load-bearing uncertainty — if features don't survive split/clone/prune coherently, the whole pipeline fails silently. Interaction-layer latency claims need real benchmarks.
validating_experiment: Build-sequence verification per [[language-grounded-3dgs-2026]] §Verification Plan. Phased implementation in [[language-grounded-3dgs-nerfstudio]].
triggers: [design-outcome:language-grounded-3dgs-2026]
design: wiki/designs/language-grounded-3dgs-2026.md
created: 2026-04-15 · updated: 2026-04-18

### Bet #022 — Shift-equivariant distillation for LangSplat's per-scene autoencoder
status: proposed
combines: [[qin2024_langsplat]], [[shift-equivariant-distillation-loss_ranzinger2026]]
stage_target: lifting-foundation-models.feature-source
op_target: op:per-scene-3dgs
confidence: low
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: LangSplat's per-scene autoencoder distills CLIP features into a per-Gaussian latent space. CLIP's output features carry positional artifacts (register-token bleed, ViTDet-border patterns inherited from SAM-mask prefix conditioning) — the same class of fixed-pattern noise that C-RADIOv4's shift-equivariant loss was designed to kill. Applying the mechanism to the LangSplat autoencoder — random patch-aligned crop-shifts between the training view and the CLIP teacher's input, loss computed on shared positions — should sharpen per-Gaussian language features at scene-geometry boundaries where CLIP's positional noise is most visible.
expected_gain: Qualitative: cleaner per-Gaussian language features on object boundaries (matching the Fig 1 improvement C-RADIOv4 shows at the image level). Quantitative: 0.5–2% 3D-OVS mIoU on LERF fine-structure queries; uncertain magnitude.
risk: The dominant failure mode of LangSplat's autoencoder is probably *compression bottleneck* (losing CLIP signal in the autoencoder latent), not positional noise. If so, shift-equivariance doesn't help — it removes a failure mode that isn't load-bearing. Worth running precisely *because* it's cheap to test: days, not weeks. Low-risk, low-confidence, high-learning-per-dollar.
validating_experiment: LangSplat with {standard per-scene autoencoder, shift-equivariant per-scene autoencoder} on LERF 3D-OVS. Qualitative inspection of per-Gaussian feature PCA at object boundaries.
triggers: [ingest-of-idea:positional-noise-in-clip-features, benchmark:langsplat-boundary-fidelity]
created: 2026-04-22 · updated: 2026-04-22

### Bet #023 — Visibility-aware 3-state voxel encoding for ConceptFusion-style online voxel feature fusion
status: proposed
combines: [[jatavallabhula2023_conceptfusion]], [[visibility-aware-masked-sparse-vae_meng2026]]
stage_target: lifting-foundation-models.online-voxel-feature-fusion
op_target: op:voxel-online
confidence: med
magnitude: incremental
cost: weeks
breakage_risk: low
hypothesis: ConceptFusion's voxel TSDF + per-voxel multimodal feature fusion currently encodes only two voxel states (observed-with-feature, no-feature-yet). It does not distinguish "observed-but-no-object-here" from "never-observed". Importing [[meng2026_seen2scene]]'s 3-state encoding (surface / observed-empty / unknown) plus the learnable empty embedding for unobserved voxels would let downstream open-vocab queries (CLIP cosine over a region, planning queries asking "is there an object here?") return *calibrated* answers — explicitly indicating "I haven't seen there" rather than silently returning low-confidence-no-object. The mechanism is generic: it's a 3-state voxel substrate, not flow-matching-specific.
expected_gain: Reduced false-positive open-vocab segmentation in occluded cavities on partial scans (e.g., under tables, inside cabinets). Quantifiable on a partial-scan held-out subset of ScanNet++ where ground truth is known but the scan didn't observe it — the metric is "did the model correctly say I-don't-know vs hallucinate".
risk: 3-state encoding adds memory (~1 bit per voxel for visibility state, plus the learnable embedding parameters). Most existing ConceptFusion benchmarks are dense / well-observed where the gain is small. Negative-result reporting ("avoid wrong answers in unobserved regions") is harder to publish than positive metric improvements.
validating_experiment: Modify ConceptFusion to use 3-state voxels + learnable empty embedding for unobserved cells. Benchmark open-vocab seg precision/recall on partial-scan subsets (mask out a held-out subset of input frames before fusion). Compare to vanilla ConceptFusion on the same subsets.
triggers: [ingest-of-paper:open-vocab-3d-seg-failure-mode-analysis-on-partial-scans]
created: 2026-04-24 · updated: 2026-04-24

### Other open questions (not yet bets)

- **Memory**: per-Gaussian features at scene scale explode memory. EA-3DGS-style codebook VQ applied to feature channels? No paper tries this — candidate for a future bet once an explicit codebook-for-semantics paper lands.
- **3D-native foundation models**: will the field invert — pretrain FMs directly on 3D rather than lift 2D features? CLIP-GS is the first step.
- **Benchmarks**: 3D-OVS and 3D instance-segmentation benchmarks remain noisy; cross-paper comparison frequently apples-to-oranges.

## Capability gaps

- **Per-primitive feature compression at scene scale** (memory-efficient lifting) — per-Gaussian features explode at city scale. Would unlock: all OPs at larger scales. Search target: codebook-VQ applied to feature channels, quantization, or hashing schemes on lifted features.
- **Densification / pruning invariants for lifted features** — current 3DGS methods handle this ad hoc; features split/merge incorrectly. Would unlock: robust training dynamics for feature-augmented 3DGS. Search target: densification rules that preserve per-primitive identity and language channels.
- **Positional-noise hygiene in lifted features** — [[shift-equivariant-distillation-loss_ranzinger2026]] shows ViT teachers inject fixed-pattern noise that students mimic; lifted 3DGS features inherit whatever noise is in the 2D feature source. No paper measures how much of per-Gaussian feature degradation is from positional noise vs. other causes. Would unlock Bet #022 and any future "clean-lift" methods. Search target: papers analyzing PCA / spectrum of per-Gaussian features vs. per-image features at object boundaries.
- **Visibility-awareness in voxel-substrate fillers** — current fillers conflate "unobserved" with "no object" (or with unfused-but-observed-empty). [[meng2026_seen2scene]] showed this is a learnable distinction worth explicit encoding for generative scene priors; cross-thread Bet #023 imports the same primitive into open-vocab feature lifting. Would unlock: calibrated open-vocab queries on partial scans. Search target: papers exposing open-vocab failure modes in occluded regions.
- **Harmonized 3D-OVS benchmark** — cross-paper comparison is noisy. Meta-gap, not a paper target.

## Contradictions & tensions

- **Per-scene vs. generalizable**: LangSplat (per-scene autoencoder) wins on per-scene quality; CLIP-GS (scene-level contrastive) wins on zero-shot transfer. No paper unifies both on the same benchmarks — the comparison is always conditioned on the scenario.
- **Representation substrate**: 3DGS (LangSplat, Gaussian Grouping) vs. sparse voxel (LangSVR, ConceptFusion voxel-TSDF) — both claim the lifting pattern, but the substrate choice shapes what downstream tasks work (NVS + editing → 3DGS; online robotics → voxel).
