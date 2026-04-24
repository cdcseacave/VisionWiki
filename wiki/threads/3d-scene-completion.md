---
title: 3D Scene Completion from Partial Real Scans
type: thread
tags: [scene-completion, generative-3d, flow-matching, tsdf, sparse-vae, controlnet, partial-data, indoor-scene]
created: 2026-04-24
updated: 2026-04-24
sources: [wiki/papers/meng2026_seen2scene.md]
operating_points: [op:default]
status: draft
---

## Goal

Take a partial 3D scan (RGB-D fusion, LiDAR sweep, or MVS-derived TSDF) of a real indoor environment and produce a complete, coherent 3D scene geometry that preserves observed surfaces while plausibly synthesizing the unobserved regions. Optional conditioning by 3D layout boxes / text / sparser keyframes. Output is volumetric (TSDF / voxel / extracted mesh) — distinct from radiance-field threads (output is geometry, not appearance) and from [[generative-3d-from-2d-priors]] (input is 3D-partial, not 2D).

The thread exists because, as of the seed paper [[meng2026_seen2scene]] (2026-04), generative completion of *real* partial 3D scans became principled for the first time. Earlier scan-completion work (SG-NN, NKSR) was either regression-style (single deterministic output, no scene-prior beyond the 3D-CNN) or constrained to synthetic-only training data (BlockFusion, LT3SD, WorldGrow). Seen2Scene's visibility-guided masking is the bridge that lets modern flow-matching/diffusion 3D generators train on the long tail of real cluttered indoor data without first requiring complete ground-truth meshes.

## Goal contract (optional, structured)

```yaml
metric: [chamfer-cm-on-observed, U3D-FPD-distribution, VLM-perceptual-score, completion-fidelity-on-held-out-frames]
target_regime: [partial-3d-scan-input, indoor-bounded-scene, static-scene]
constraints: [observed-region-preservation, no-complete-GT-required-for-training, generative-not-regression]
required_capabilities: [3d-scene-prior-trainable-on-partial-data, visibility-aware-encoder, observed-region-preservation-during-generation]
```

## SOTA pipelines

Single OP at thread inception. The OP may split when (a) a paper closes a different point on the speed-vs-quality or geometry-only-vs-with-appearance trade-off, or (b) the use case "completion of an existing partial scan" diverges from "from-scratch text/layout-driven scene generation" enough to warrant separate pipelines.

### op:default (offline, indoor, layout-optional, geometry-only)

The pipeline is a DAG of nodes. Each node has a thread-local stable ID (n1, n2, ...); IDs never reused.

- **n1** · stage(s): `[[scene-completion.partial-scan-latent-encoding]]` · filler: [[visibility-aware-masked-sparse-vae_meng2026]] · source: [Meng et al. 2026](../papers/meng2026_seen2scene.md) · gain over prior: Table 3 reconstruction column — without masked training, L1/L2/CD all degrade significantly · upstream: raw partial TSDF (256³, 1.1 cm voxels, sentinel-encoded unobserved) from VDBFusion-style volumetric fusion · downstream: [n2, n3] · edge types: `<in: partial-tsdf-3-state; out: latent-grid-32-cubed-8-channel + visibility-mask>`.

- **n2** · stage(s): `[[scene-completion.generative-prior]]` · filler: [[visibility-guided-masked-flow-matching_meng2026]] · source: [Meng et al. 2026](../papers/meng2026_seen2scene.md) · gain over prior: Table 3 generation column — without masked training, U3D-FPD degrades and VLM perceptual score drops; vs unconditional baselines (LT3SD, WorldGrow) wins on Table 2 (VLM avg 5.70 vs prior best LT3SD); vs layout-conditional (BlockFusion) wins on FID + FPD · upstream: [n1] (latent + mask) + [n3] (layout tokens) · downstream: [n4]. **Bundled with n1** (visibility-mask consumer requires visibility-aware encoder).

- **n3** · stage(s): `[[scene-completion.generative-prior]]` (conditioning sub-component) · filler: [[clip-painted-3d-layout-conditioning_meng2026]] · source: [Meng et al. 2026](../papers/meng2026_seen2scene.md) · gain over prior: Table 4 — CLIP semantic encoding significantly outperforms one-hot category embedding on real datasets (ScanNet++, ARKitScenes); enables cross-dataset training over 4,489 unique categories · upstream: external layout source (artist input / LLM / 2D-to-layout / 3D-detector) · downstream: [n2]. Sub-stage of n2 in this pipeline; could also fill an independent `scene-completion.layout-conditioning` stage if/when other generators want layout but not the masked-flow-matching backbone.

- **n4** · stage(s): `[[scene-completion.condition-injection]]` · filler: [[controlnet-frozen-flow-self-supervised-completion_meng2026]] · source: [Meng et al. 2026](../papers/meng2026_seen2scene.md) · gain over prior: Tables 1 + 2 vs SG-NN [Dai 2020] and NKSR [Huang 2023] — Seen2Scene wins on completion CD/L2/U3D-FPD; non-zero TMD (multi-sample diversity) where baselines are deterministic · upstream: [n2] (frozen pretrained generator) + query partial scan via [n1] (frozen VAE) · downstream: TSDF volume → marching-cubes mesh extraction (could feed [[mesh-reconstruction.extraction]] cross-thread).

**Node bundle note**: n1 ↔ n2 is a hard `co_requires:` (the visibility mask is meaningful only if downstream consumes it). n4 requires both n1 and n2 architecturally (ControlNet branch is initialized from n2's weights; condition encoder shares n1).

## Pipeline lineage

- 2026-04-24 · op:default · **initial pipeline** (scope: new-paradigm) · nodes introduced: [n1, n2, n3, n4] · driver: [[meng2026_seen2scene]] · rationale: thread inception. Visibility-guided flow matching is the first principled mechanism for training generative 3D scene priors directly on partial real-world scans, distinct from synthetic-data-only generators (BlockFusion / LT3SD / WorldGrow) and from regression-style completion (SG-NN / NKSR). The four nodes form a tightly-bundled stack — none of the ideas have known viable substitutes in the wiki yet, so the entire pipeline is single-source.

## Candidate components / not yet integrated

(none yet — thread inception. Future ingests will populate this with ideas that lose Pass A on this thread but are kept as candidates.)

## Open questions & synthesis bets

The synthesis pressure on a brand-new thread is naturally high — there's only one paper backing the SOTA pipeline, so most "open question" entries here are search targets rather than novel-combination bets.

### Bet #001 — Visibility-aware voxel encoding for online voxel feature fusion (cross-thread to lifting-foundation-models-to-3d)

status: proposed
combines: [[visibility-aware-masked-sparse-vae_meng2026]], [[voxel-tsdf-multimodal-fusion_jatavallabhula2023]]
stage_target: lifting-foundation-models.online-voxel-feature-fusion
op_target: op:default (this thread); cross-thread to lifting-foundation-models-to-3d
confidence: med
magnitude: incremental
cost: weeks
breakage_risk: low
hypothesis: ConceptFusion-style online voxel TSDF feature fusion currently treats unobserved voxels and unfused-but-observed voxels identically (both have no feature). Importing Seen2Scene's 3-state encoding (surface / observed-empty / unknown) lets downstream consumers (foundation-model lookups, segmentation queries, planning) distinguish "I haven't seen here" from "I saw here but no object". This should improve open-vocab segmentation precision on cluttered scenes by reducing false-positive feature lookups in unobserved cavities.
expected_gain: ConceptFusion-style open-vocab 3D mIoU on ScanNet++ partial scans with explicit unobserved-region exclusion; first method to factor visibility into multimodal voxel queries.
risk: 3-state encoding adds memory + compute; may not be worth it on dense well-observed scenes. Also the gain is mostly negative ("avoid wrong answers in unobserved regions") which is harder to demonstrate than a positive metric improvement.
validating_experiment: Modify ConceptFusion's voxel-feature fusion to use a learnable empty embedding for unobserved voxels; benchmark open-vocab seg precision/recall on partial-scan held-out subsets.
triggers: [ingest-of-paper:open-vocab-3d-seg-failure-mode-analysis-on-partial-scans, ingest-of-conceptfusion-followup]
created: 2026-04-24 · updated: 2026-04-24

### Bet #002 — Mono-depth → partial TSDF → Seen2Scene completion (cross-thread to mono-depth-estimation)

status: proposed
combines: [[visibility-guided-masked-flow-matching_meng2026]], [[visibility-aware-masked-sparse-vae_meng2026]], [[controlnet-frozen-flow-self-supervised-completion_meng2026]], (any-mono-depth-estimator)
stage_target: scene-completion.condition-injection
op_target: op:default (this thread); cross-thread to mono-depth-estimation as a downstream consumer
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: Mono-depth predictors (DepthAnythingv3, Metric3Dv2) emit per-pixel depth from a single RGB image. Fusing a few mono-depth predictions into a TSDF (with explicit visibility tracking from the synthetic camera) produces a partial scan from a single image — without any actual depth sensor. Feeding this into Seen2Scene completion turns *any RGB image* into a complete 3D scene via the generative prior. This is a different bridge from [[generative-3d-from-2d-priors]] (single-image latent flow matching for objects) — it routes through a *visibility-aware partial 3D scan* intermediate, which preserves geometric structure that single-image latent generators discard.
expected_gain: Single-image to scene-scale completed 3D, with explicit observed-vs-hallucinated separation (the visibility mask). Useful for scene editing and for sparse-view 3DGS initialization.
risk: Mono-depth scale + alignment errors compound through TSDF fusion; the resulting partial scan may be too noisy or too biased for the visibility-aware encoder to handle gracefully (the encoder was trained on RGB-D / LiDAR-fused TSDFs with sensor-grade depth quality). Also bidirectional: the paper limits to indoor; outdoor mono-depth → outdoor scene completion would require retraining Seen2Scene on outdoor data.
validating_experiment: Take ScanNet++ RGB images, run mono-depth, fuse to partial TSDF with visibility tracking, run Seen2Scene completion, compare to native-RGB-D-fused completion. Quantify the mono-depth-vs-sensor gap on completion fidelity.
triggers: [ingest-of-paper:mono-depth-to-tsdf-fusion-benchmark, ingest-of-papers:depthanythingv3-or-metric3dv3]
created: 2026-04-24 · updated: 2026-04-24

### Bet #003 — Lyra-2.0 video-world geometry → Seen2Scene completion (cross-thread to generative-3d-from-2d-priors)

status: proposed
combines: [[per-frame-3d-cache-retrieval_shen2026]], [[downsampled-gaussian-dpt-head_shen2026]], [[controlnet-frozen-flow-self-supervised-completion_meng2026]], [[visibility-guided-masked-flow-matching_meng2026]]
stage_target: scene-completion.condition-injection
op_target: op:default (this thread); cross-thread to generative-3d-from-2d-priors:op:explorable-scene
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: Lyra 2.0 generates an explorable 3DGS world from a single image but the resulting geometry is constrained by the camera trajectory it generated and exhibits artifacts (DL3DV exposure variation propagation per Lyra 2.0 §6). Lifting that 3DGS to a partial TSDF (via per-Gaussian depth fusion) and running Seen2Scene completion could repair occluded regions (parts of the scene the generated walkthrough never visited) and produce a globally coherent volumetric output — bridging the video-world `op:explorable-scene` to a TSDF/mesh `op:default`-style output. This is the mirror image of Bet #2 (which routes mono-depth → TSDF) — here the 2D-image → 3D path goes through Lyra's video-world before fusing to TSDF.
expected_gain: Combined pipeline: explorable + complete 3D scene from single image. Closes the video-world coverage gap (only what the generated camera trajectory visited gets explicit geometry) using a generative completion prior rather than re-trajectory-generating to fill gaps.
risk: Lyra 2.0's 3DGS has its own coordinate frame + photometric biases; fusing to partial TSDF then completing introduces compound failure modes. Latency is also problematic — single-image → video-world → 3DGS-extract → TSDF-fuse → Seen2Scene-complete is the slowest possible inference path.
validating_experiment: Take a held-out single-image input, run Lyra 2.0 op:explorable-scene end-to-end, fuse the resulting 3DGS to partial TSDF with visibility tracking from the generated camera trajectory, run Seen2Scene completion, evaluate geometric completeness vs Lyra-only and vs Seen2Scene-on-mono-depth-of-same-image (Bet #2).
triggers: [ingest-of-paper:gaussian-to-tsdf-with-visibility, real-world-deployment-latency-acceptable]
created: 2026-04-24 · updated: 2026-04-24

### Bet #004 — Generative scene completion as a sparse-view 3DGS post-processor (cross-thread to gaussian-to-mesh-pipelines)

status: proposed
combines: [[controlnet-frozen-flow-self-supervised-completion_meng2026]], [[delaunay-mesh-in-loop_guedon2025]]
stage_target: mesh-reconstruction.extraction (cross-thread)
op_target: op:default (this thread); cross-thread to gaussian-to-mesh-pipelines
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: Sparse-view 3DGS reconstructions produce meshes with large holes in occluded regions (areas not visible from any input view). [[meng2026_seen2scene]]'s ControlNet completion accepts a partial TSDF — fusing the sparse-view 3DGS depth into a partial TSDF and running completion should fill the holes plausibly. The frame-drop self-supervised pretext is exactly the right training signal: the gap between "what a few views see" and "what the full scene contains" is structurally similar to the (more-partial, less-partial) pairs the model was trained on.
expected_gain: Holes-filled sparse-view 3DGS meshes; usable for asset creation from few-view captures. Quantifiable on a sparse-view benchmark (1-5 input views) by completion fidelity at held-out test views.
risk: Sparse-view 3DGS depth estimates are noisy; partial TSDFs derived from them may be too biased for the visibility-aware encoder to handle. Also, completion fills holes plausibly but not photometrically — the completed regions have no color/texture, requiring a second pass for appearance.
validating_experiment: Pipeline: sparse-view 3DGS → render depth from input views → fuse to partial TSDF → Seen2Scene complete → marching cubes for mesh. Compare hole-completeness + chamfer-on-held-out-views to MILo and to Gaussian Surfels.
triggers: [ingest-of-paper:sparse-view-3dgs-mesh-extraction-benchmark]
created: 2026-04-24 · updated: 2026-04-24

## Capability gaps

- **Outdoor-scale generalization** — Seen2Scene is trained indoor-only; no current filler covers outdoor (cars, vegetation, building exteriors). Would unlock: SLAM/SfM-derived sparse outdoor scans → completed outdoor scenes for autonomous-driving / aerial-photogrammetry. Search target: a paper applying visibility-guided masked flow matching to outdoor LiDAR / aerial imagery, OR an outdoor 3D-FRONT equivalent.

- **Resolution ceiling** — 1.1 cm voxel + 8× VAE downsampling is the current limit; fine surface detail (chair-leg edges, fabric folds) is lost. Would unlock Bet #4 above (sparse-view 3DGS post-processing needs millimeter-class detail to be useful for mesh asset creation). Search target: hierarchical-VAE or higher-res sparse grid for generative 3D, or a non-VAE encoder.

- **Single-pose-per-voxel assumption (static-scene)** — visibility tracking assumes a single observation history per voxel. Dynamic objects would require temporal-aware visibility. Would unlock: 4D scene completion. Search target: a paper extending visibility-guided generation to dynamic-object scenes — possibly cross-pollinating with 4D Gaussians threads.

- **No appearance / material output** — pure geometry. Would unlock: turn completed scenes into ready-to-deploy game / sim assets. Search target: a paper combining flow-matching geometry generation with a per-voxel appearance prior, or a two-stage geometry → texture pipeline.

- **Patch-boundary inconsistencies for house-scale** — independent 256³ patch completion + weighted-average overlap fusion. Long-range structures (hallways, beams) may show seams. Would unlock: scalable to building-scale without global re-optimization. Search target: hierarchical or globally-attended sparse generation, or a patch-aware fusion strategy.

- **Sub-component ablation gap** — the paper does not separately ablate ControlNet vs frame-drop in [[controlnet-frozen-flow-self-supervised-completion_meng2026]]. Cross-thread bets that import only one component (e.g., frame-drop pretext into a different completion model) cannot weight the bet without this. Would unlock: cleaner cross-thread transfer of the completion mechanism. Search target: a re-implementation or follow-up paper with the missing ablation.

- **Generative scene priors for non-TSDF representations** — the visibility-mask trick is TSDF-bound (sentinel encoding). Point-cloud / Gaussian-splat / mesh-native equivalents are unexplored. Would unlock: visibility-aware generative priors compatible with downstream consumers that don't speak TSDF. Search target: a paper extending masked-flow-matching to point clouds or to native Gaussian splats.

## Contradictions & tensions

- **Generative completion vs deterministic reconstruction** — Seen2Scene's stochasticity (non-zero TMD = topological mesh diversity) is positioned as a feature vs SG-NN/NKSR's deterministic outputs. But for some downstream consumers (asset creation, simulation) determinism is preferred. Open: should the SOTA filler always be the stochastic generator, or is there a `op:deterministic` worth carving out for use cases where reproducibility matters more than diversity?

- **Layout vs partial-scan condition weighting** — Table 1 of the paper hints that adding bounding boxes slightly degrades L2 on observed surfaces (the model trusts layout enough to over-write observed geometry). Open: should the ControlNet branch be weighted higher in observed regions? No principled answer in the paper.

## Shelved bets / known non-compositions

(none yet — thread is brand new.)

## Sources

- [meng2026_seen2scene.md](../papers/meng2026_seen2scene.md) — **op:default driver**. Sole source for n1, n2, n3, n4. Visibility-guided masked flow matching paradigm; first generative 3D scene model trained directly on partial real scans.
