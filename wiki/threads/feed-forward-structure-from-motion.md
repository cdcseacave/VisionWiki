---
title: Feed-Forward Structure from Motion
type: thread
tags: [sfm, pose-estimation, feed-forward, dust3r, mast3r, rommav2, transformer, test-time-training]
created: 2026-04-12
updated: 2026-04-21
sources: [papers/zhong2026_instantsfm.md, papers/pataki2025_mp-sfm.md, papers/yu2025_cusfm.md, papers/murai2025_mast3r-slam.md, papers/li2025_megasam.md, papers/zhao2025_diffusionsfm.md, papers/zhang2025_loger.md, papers/jin2026_zipmap.md, papers/zhang2024_cameras-as-rays.md, papers/jang2025_pow3r.md, papers/edstedt2025_roma-v2.md, papers/zhang2025_feed-forward-3d-survey.md, papers/chen2026_ttt3r.md, papers/he2023_detector-free-sfm.md, papers/yu2025_madpose.md]
operating_points: [op:default]
status: draft
---

## Goal

Replace the classical SfM pipeline (detect → match → verify → triangulate → BA) with learned components, ideally in a single feed-forward pass, while matching COLMAP-grade pose accuracy and scaling to long sequences (hundreds to thousands of frames). The thread tracks the three-tier split: accelerated-classical (see [[gpu-native-sfm]]), classical+learned priors, fully feed-forward.

## Goal contract (optional, structured)

```yaml
metric: [pose-AUC@5deg, ATE, chamfer, throughput-fps, sequence-length-frames]
target_regime: [unposed-images, sparse-to-dense views, static | dynamic scenes]
constraints: [calibration-grade-pose-floor, linear-time-in-sequence-length]
required_capabilities: [pose-regression, pointmap-prediction, long-context-scaling]
```

## Capability gaps

- **Calibration-grade outdoor accuracy from fully feed-forward methods** — no Tier-3 paper matches COLMAP on outdoor benchmarks without a BA tail. Search target: 2026 scale-up of DUSt3R/VGGT-family on outdoor training data.
- **Dynamic-scene handling without heuristic motion masks** — MegaSaM is the best attempt but still relies on learned segmentation. Search target: end-to-end dynamic SfM where motion is inferred implicitly.
- **Principled fusion of ≥3 learned priors** (depth + normal + correspondence + flow) into classical BA — MP-SfM uses depth+normal; InstantSfM uses depth; no paper fuses all three. All three atomic priors are now in the wiki as `drop-in` / `stage-swap` ideas: [[depth-proportional-uncertainty-fusion_pataki2025]] + [[bilateral-normal-integration-with-uncertainty_pataki2025]] + [[roma-v2-predictive-covariance_edstedt2025]]. The gap is no longer acquiring the priors — it is the combinatorial ablation. [[gpu-native-sfm]] Bet #010 is the scheduled experiment; flow remains an unaddressed fourth modality. Search target: ablation-heavy multi-prior-fusion papers that include optical flow alongside depth/normal/match-confidence.
- **Texture-poor + low-overlap joint handling** — [[he2023_detector-free-sfm|DetectorFreeSfM]] closes texture-poor; [[pataki2025_mp-sfm|MP-SfM]] closes low-overlap. Each paper tests only its own failure mode; neither tests the intersection. Search target: future benchmark that combines both regimes (e.g., Texture-Poor SfM dataset + MP-SfM's 0%-overlap ETH3D split). Resolved by: Bet #019 below.
- **Pair-pose quality as SfM frontend** — MADPose shows +15–27 AUC@10° pair-pose gains but no paper has integrated it into multi-view SfM init. The cross-pair shift-consistency problem is the integration blocker. Search target: papers solving global mono-depth-affine consistency across image sequences. Cross-referenced from [[relative-pose-estimation]] Bet #016.

## Working hypothesis

The SfM landscape is splitting into three tiers:

1. **Accelerated classical** (InstantSfM, CuSfM): keep COLMAP's math, rewrite
   it for GPUs. 40x speedups, same accuracy. Low risk, high practical value.
2. **Hybrid classical + learned priors** (MP-SfM, MegaSaM, Pow3R): inject mono
   depth/normal priors into the classical pipeline. Extends SfM to harder cases
   (sparse views, dynamic scenes) while keeping optimization-based refinement.
3. **Fully feed-forward** (DiffusionSfM, LoGeR, ZipMap, MASt3R-SLAM): replace
   the pipeline with a single neural forward pass. Highest ceiling, least mature.

The DUSt3R/MASt3R family is the backbone of tier 3. The frontier is scaling
feed-forward methods to long sequences (hundreds/thousands of frames) without
quadratic attention costs.

## Evidence

### Tier 1: GPU-accelerated classical SfM
- [InstantSfM (Zhong 2026)](../papers/zhong2026_instantsfm.md): full global SfM
  in PyTorch. Depth-constrained Jacobians, GPU-native BA. 40x faster than
  [[colmap|COLMAP]] on 1K+ image datasets. The "just make COLMAP fast" approach.
- [CuSfM (Yu 2025)](../papers/yu2025_cusfm.md): CUDA SfM tailored for
  autonomous driving — multi-camera rig support, extrinsic refinement. Practical
  engineering over novelty.
- **Deep dive**: see [[gpu-native-sfm]] thread for the full Tier 1 story,
  including the shared playbook and InstantSfM-vs-CuSfM trade-offs.

### Tier 2: Classical + learned priors
- [MP-SfM (Pataki 2025)](../papers/pataki2025_mp-sfm.md): mono depth + normal
  priors from [[Metric3Dv2]] eliminate the three-view overlap requirement in
  incremental SfM. Key result: SfM works on sparse drone/indoor imagery where
  COLMAP fails. The full-system idea
  ([[mono-depth-normal-constrained-incremental-sfm_pataki2025]]) is a pipeline
  DAG change (new init/registration/filter nodes replacing classical ones),
  not a single-stage swap — adopters take the DAG change and three
  independently-reusable atoms together:
  next-view selection by summed matcher score
  ([[matcher-score-next-view-selection_pataki2025]]),
  mono-prior uncertainty calibration
  ([[depth-proportional-uncertainty-fusion_pataki2025]]), and BA with
  bilateral normal integration under propagated covariance
  ([[bilateral-normal-integration-with-uncertainty_pataki2025]]).
- [MegaSaM (Li 2025)](../papers/li2025_megasam.md): learned motion segmentation
  + uncertainty-aware BA enables SfM from casual dynamic videos (people walking,
  cars moving). Bridges the "static scene assumption" gap.
- [Pow3R (Jang 2025)](../papers/jang2025_pow3r.md): lightweight conditioning
  injects optional camera intrinsics/poses/sparse depth into [[dust3r|DUSt3R]]. Shows
  that the DUSt3R architecture can absorb classical priors when available.
- [DetectorFreeSfM (He 2023)](../papers/he2023_detector-free-sfm.md):
  detector-free semi-dense matcher (LoFTR) drives classical COLMAP mapping
  via a match-quantization bridge + multi-view transformer track refinement
  + iterative BA / track-topology adjustment. **Closes the texture-poor
  failure mode** of classical SfM with geometric-BA memory 340× smaller
  than PixSfM. Three co-required ideas travel together:
  [[coarse-to-fine-detector-free-sfm-bridge_he2023]],
  [[multi-view-transformer-track-refinement_he2023]],
  [[iterative-ba-plus-track-topology-adjustment_he2023]]. Orthogonal failure
  mode to MP-SfM's low-overlap — the two compose (proposed in Bet #017 of
  [[gpu-native-sfm]], combo also flagged below).
- [MADPose (Yu 2025)](../papers/yu2025_madpose.md): depth-aware two-view
  pose estimator (affine-corrected 3pt/4pt solvers + hybrid LO-MSAC).
  Pair-pose only — the full thread-level coverage lives in
  [[relative-pose-estimation]]. Relevant to Tier 2 as a better
  view-graph-construction frontend; [[hybrid-lo-msac-dual-modality-estimator_yu2025]]
  bundle-linked with [[affine-corrected-minimal-relative-pose-solvers_yu2025]]
  and [[depth-induced-reprojection-scoring_yu2025]].

### Tier 3: Fully feed-forward
- [MASt3R-SLAM (Murai 2025)](../papers/murai2025_mast3r-slam.md): first
  real-time dense SLAM on [[mast3r|MASt3R]] priors. 15 FPS, no calibration needed.
  Proves the DUSt3R family can serve as a live SLAM backend.
- [DiffusionSfM (Zhao 2025)](../papers/zhao2025_diffusionsfm.md): diffusion
  model predicts ray origins + endpoints for joint structure and motion. Novel
  formulation — no explicit pose parameterization at all.
- [LoGeR (Zhang 2025)](../papers/zhang2025_loger.md): hybrid TTT + sliding
  window attention scales to 19K frames. Key innovation: [[test-time-training]]
  as a linear-time memory mechanism for long sequences.
- [ZipMap (Jin 2026)](../papers/jin2026_zipmap.md): bidirectional TTT fast
  weights, 20x faster than [[vggt|VGGT]]. Shows test-time-training is becoming the
  dominant paradigm for scaling feed-forward reconstruction.
- [TTT3R (Chen 2026)](../papers/chen2026_ttt3r.md): the *training-free*
  variant of the same idea. Reinterprets [[CUT3R]]'s recurrent state as a
  fast weight and derives a **closed-form confidence-guided learning rate**
  from the existing attention alignment — no retraining, no architecture
  change. 2× pose improvement over CUT3R on long sequences at 20 FPS / 6 GB
  for thousands of images. Distinguishes itself from LoGeR/ZipMap by being
  a plug-and-play patch to the recurrence rather than a new trained model,
  and articulates the unified `Tokenize → Update → Read → De-tokenize`
  formulation that separates full-attention (VGGT, Fast3R) from RNN
  (CUT3R, Point3R) pointmap models cleanly.
- [Cameras as Rays (Zhang 2024)](../papers/zhang2024_cameras-as-rays.md):
  Plucker ray-bundle camera representation + diffusion for sparse-view pose.
  Earlier work that planted the "cameras are just rays" idea.

### Cross-cutting: matching and features
- [RoMa v2 (Edstedt 2025)](../papers/edstedt2025_roma-v2.md): dense matcher
  with DINOv3 backbone and custom CUDA refinement. Even in feed-forward SfM,
  correspondence quality still matters — RoMa v2 is the current matching SOTA.
- [Survey (Zhang 2025)](../papers/zhang2025_feed-forward-3d-survey.md):
  comprehensive taxonomy confirming the three-tier split above. Identifies
  pointmap prediction (DUSt3R-style) as the dominant feed-forward paradigm.

## Emerging patterns

- **Test-time training** (LoGeR, ZipMap, TTT3R) is the scaling trick for feed-forward
  reconstruction: linear time, no quadratic KV cache, handles thousands of frames.
  Frontier question: does TTT need retraining (LoGeR, ZipMap) or can it be
  derived from the frozen model's own attention (TTT3R)? TTT3R's training-free
  closed-form learning rate is the cheapest answer yet.
- **DUSt3R/MASt3R** is the gravitational center — Pow3R extends it, MASt3R-SLAM
  runs it in real-time, DiffusionSfM rethinks it.
- **Monocular depth priors** are the bridge between classical and learned: MP-SfM
  and InstantSfM both use them to make classical BA more robust.

## Open questions
- Can feed-forward methods match COLMAP accuracy on large-scale outdoor scenes?
  InstantSfM matches it by being COLMAP-on-GPU; the fully learned methods
  (DiffusionSfM, ZipMap) haven't proven this yet.
- Is the endgame a single model (images → posed point cloud) or hybrid?
  Evidence leans hybrid — even LoGeR and ZipMap use DINOv2 backbones, not
  end-to-end training.
- How sensitive are feed-forward methods to domain shift? Most train on
  ScanNet/CO3D — outdoor generalization is under-tested.
- Will test-time-training replace attention for long-context 3D?

## Related threads
- [[radiance-field-evolution]]
- [[mono-depth-estimation]]
- [[relative-pose-estimation]] — depth-aware two-view pose as a standalone problem (thread split off 2026-04-21 when ingesting [[yu2025_madpose|MADPose]])
- [[gaussian-to-mesh-pipelines]] — VGG-T3 bridges feed-forward to mesh output

## Pass B synthesis bets (cross-paper)

### Bet #019 — DetectorFreeSfM × MP-SfM compose (texture-poor AND low-overlap simultaneously)
status: proposed
combines: [[coarse-to-fine-detector-free-sfm-bridge_he2023]], [[multi-view-transformer-track-refinement_he2023]], [[iterative-ba-plus-track-topology-adjustment_he2023]], [[mono-depth-normal-constrained-incremental-sfm_pataki2025]]
stage_target: full SfM pipeline (feature-matching + bootstrap + registration + BA)
op_target: [[feed-forward-structure-from-motion]] Tier 2
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: DetectorFreeSfM replaces the matching frontend with a detector-free matcher + quantization bridge (solving texture-poor). MP-SfM replaces the bootstrap/registration/BA with mono-depth-driven variants (solving low-overlap). The two failure modes are orthogonal and their solutions operate at disjoint stages — compose them: LoFTR-based quantized matches feed MP-SfM's incremental pipeline (which now lifts points via mono-depth for bootstrap, does depth-constrained BA, and forward-backward-depth-consistency filters). Target: scenes that are **both** texture-poor **and** low-overlap (drone captures of low-textured surfaces, underwater at wide baselines).
expected_gain: first end-to-end SfM that handles both regimes simultaneously; new benchmark numbers on a dataset combining texture-poor + low-overlap. No existing benchmark targets this; dataset creation is part of the bet.
risk: MP-SfM's mono-depth lifting assumes the bootstrap pair has enough textureless matches to fit the depth scale — texture-poor might starve the matching even after quantization merging. Mitigation: fall back to MP-SfM's PnP-from-depth bootstrap when quantized-match count is below threshold.
validating_experiment: build a combined benchmark (Texture-Poor SfM + 0%-overlap ETH3D + drone low-texture captures); run {COLMAP, DetectorFreeSfM, MP-SfM, combo} on all; compare AUC and completeness.
triggers: [ingest-of-idea:coarse-to-fine-detector-free-sfm-bridge_he2023, ingest-of-idea:mono-depth-normal-constrained-incremental-sfm_pataki2025]
created: 2026-04-21 · updated: 2026-04-21

### Bet #020 — DINOv3-backboned detector-free matching (LoFTR → RoMa v2) into DetectorFreeSfM
status: proposed
combines: [[coarse-to-fine-detector-free-sfm-bridge_he2023]], [[roma-v2-dinov3-dense-matcher_edstedt2025]]
stage_target: feature-matching.task-head
op_target: [[feed-forward-structure-from-motion]] Tier 2
confidence: high
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: DetectorFreeSfM's quantization bridge is matcher-agnostic; the paper tests LoFTR, AspanFormer, MatchFormer — all 2021–2022 models. RoMa v2 on DINOv3 is the current dense-matching SOTA ([[foundation-features-for-geometry]]) and produces substantially more accurate coarse-stride matches. Swapping LoFTR → RoMa v2 inside the bridge should improve every downstream number without changing the pipeline.
expected_gain: +2–5 AUC@10° on ETH3D and IMC; larger gains expected on texture-poor where DINOv3 features help more than LoFTR's CNN features.
risk: RoMa v2's coarse stride differs from LoFTR's — may require re-tuning `r` in the bridge.
validating_experiment: replace LoFTR with RoMa v2 in the DetectorFreeSfM public code; re-run ETH3D/IMC/Texture-Poor.
triggers: [ingest-of-idea:coarse-to-fine-detector-free-sfm-bridge_he2023, ingest-of-idea:roma-v2-dinov3-dense-matcher_edstedt2025]
created: 2026-04-21 · updated: 2026-04-21

