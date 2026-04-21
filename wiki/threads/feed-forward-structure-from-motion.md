---
title: Feed-Forward Structure from Motion
type: thread
tags: [sfm, pose-estimation, feed-forward, dust3r, mast3r, rommav2, transformer, test-time-training]
created: 2026-04-12
updated: 2026-04-21
sources: [papers/zhong2026_instantsfm.md, papers/pataki2025_mp-sfm.md, papers/yu2025_cusfm.md, papers/murai2025_mast3r-slam.md, papers/li2025_megasam.md, papers/zhao2025_diffusionsfm.md, papers/zhang2025_loger.md, papers/jin2026_zipmap.md, papers/zhang2024_cameras-as-rays.md, papers/jang2025_pow3r.md, papers/edstedt2025_roma-v2.md, papers/zhang2025_feed-forward-3d-survey.md, papers/wang2026_feed-forward-3d-scene-modeling.md, papers/chen2026_ttt3r.md, papers/he2023_detector-free-sfm.md, papers/yu2025_madpose.md, papers/leroy2024_mast3r.md, papers/shen2025_fastvggt.md, papers/wang2025_faster-vggt-block-sparse.md, papers/feng2025_quantvggt.md, papers/shen2026_lyra2.md]
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

- **Calibration-grade outdoor accuracy from fully feed-forward methods** — no Tier-3 paper matches COLMAP on outdoor benchmarks without a BA tail. **Partially closed at the pair-pose level by [[leroy2024_mast3r|MASt3R]]** (ingested 2026-04-21): [[metric-scale-pointmap-loss_leroy2024]] gives 94.1 VCRE AUC / 0.42m median translation on Map-free test from a single forward pass (direct regression on the pointmap, no matching, no RANSAC). The remaining gap is **sequence-level**: MASt3R predicts per-pair metric pointmaps that are internally consistent but don't automatically compose into a globally-consistent multi-view reconstruction (scale mismatch across pairs, Sim(3) alignment still required). Search target: Tier-3 papers that propagate MASt3R-style metric output across long sequences without BA.
- **Dynamic-scene handling without heuristic motion masks** — MegaSaM is the best attempt but still relies on learned segmentation. Search target: end-to-end dynamic SfM where motion is inferred implicitly.
- **Principled fusion of ≥3 learned priors** (depth + normal + correspondence + flow) into classical BA — MP-SfM uses depth+normal; InstantSfM uses depth; no paper fuses all three. All three atomic priors are now in the wiki as `drop-in` / `stage-swap` ideas: [[depth-proportional-uncertainty-fusion_pataki2025]] + [[bilateral-normal-integration-with-uncertainty_pataki2025]] + [[roma-v2-predictive-covariance_edstedt2025]]. The gap is no longer acquiring the priors — it is the combinatorial ablation. [[gpu-native-sfm]] Bet #010 is the scheduled experiment; flow remains an unaddressed fourth modality. Search target: ablation-heavy multi-prior-fusion papers that include optical flow alongside depth/normal/match-confidence.
- **Texture-poor + low-overlap joint handling** — [[he2023_detector-free-sfm|DetectorFreeSfM]] closes texture-poor; [[pataki2025_mp-sfm|MP-SfM]] closes low-overlap. Each paper tests only its own failure mode; neither tests the intersection. Search target: future benchmark that combines both regimes (e.g., Texture-Poor SfM dataset + MP-SfM's 0%-overlap ETH3D split). Resolved by: Bet #019 below.
- **Pair-pose quality as SfM frontend** — MADPose shows +15–27 AUC@10° pair-pose gains but no paper has integrated it into multi-view SfM init. The cross-pair shift-consistency problem is the integration blocker. Search target: papers solving global mono-depth-affine consistency across image sequences. Cross-referenced from [[relative-pose-estimation]] Bet #016.
- **View-selection-bias in standard benchmarks** — [[wang2026_feed-forward-3d-scene-modeling]] §7.1 observes that RealEstate10K / ACID / most Tier-3 training benchmarks use fixed view splits that let models game specific viewpoint patterns, and few provide 3D ground truth. The thread's "outdoor generalization under-tested" concern (see Open questions) was implicitly about domain shift; Wang 2026 reframes it as a protocol-design problem — the benchmarks don't stratify by viewpoint-gap difficulty. Search target: benchmark papers that explicitly vary baseline / overlap / contextual-gap as a difficulty axis and report 3D-ground-truth-backed accuracy.
- **VGGT-efficiency sub-family for long-context feed-forward pointmap prediction** — ~~[[wang2026_feed-forward-3d-scene-modeling]] §4.3 flags a parallel efficiency direction to the TTT story... None are in the wiki.~~ **Partially closed 2026-04-21 by ingesting all three papers.** The family now lives in the wiki as four ideas across three orthogonal resource axes: [[vggt-token-merge-3-part-partition_shen2025]] (token count), [[pooled-qk-block-sparse-global-attention_wang2025]] (attention sparsity), [[vggt-dual-smoothed-quantization_feng2025]] + [[frame-aware-ptq-calibration-sampling_feng2025]] (numerical precision, bundled). The **open portion of the gap** is the head-to-head benchmark vs. the TTT-scaling family ([[loger-hybrid-ttt-plus-swa_zhang2025]], [[large-chunk-ttt-fast-weights_jin2026]], [[ttt3r-closed-form-confidence-lr_chen2026]], [[ttt-mlp-kv-compression_elflein2026]]): none of the six papers compare to the other branch on a common long-sequence benchmark. Bet #030 is the natural experiment. Additional search target: a VGGT-compression paper that includes 500+ frame pose ATE numbers directly comparable to TTT3R's — currently inferred indirectly through VGGT-as-baseline comparisons.
- **Reconstruction-vs-generation spectrum now has its canonical video-world exemplar** — [[wang2026_feed-forward-3d-scene-modeling]] §7.6 introduced the spectrum; [[shen2026_lyra2|Lyra 2.0]] is now the in-wiki *generation-first, reconstruction-second* anchor (generate video from a single image via bi-directional DiT, then feed-forward lift via DAv3 + fine-tune). The relevance to this thread is that Lyra 2.0's [[per-frame-3d-cache-retrieval_shen2026]] is a pure *routing* mechanism that could in principle serve the long-context-scaling problem on this thread's side — selecting which frames enter a TTT-family fast-weight update by 3D visibility rather than temporal proximity. Currently no in-wiki SfM paper does this; [[wang2026_feed-forward-3d-scene-modeling]] §4.5.1's LongStream (gauge-decoupled streaming) is the closest adjacent mechanism. Search target: a feed-forward SfM paper that applies visibility-based frame selection to long-horizon pose/pointmap prediction; alternatively, a demonstration that Lyra 2.0-style retrieval ports to the Tier-3 reconstruction setting (Bet #027 candidate — currently filed on the primary thread as it involves cross-thread composition).

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
- [MASt3R (Leroy 2024)](../papers/leroy2024_mast3r.md): the foundational
  Tier-3 matching-aware paper. Augments [[dust3r|DUSt3R]] with a dense
  local-descriptor head ([[dust3r-matching-head_leroy2024]]) trained
  jointly via InfoNCE, and adds a metric-scale loss variant
  ([[metric-scale-pointmap-loss_leroy2024]]) on the 10 metric datasets
  in its 14-dataset training mix. Introduces two mechanism-independent
  utilities: [[fast-reciprocal-nn-matching_leroy2024]] (O(kWH) vs
  O(W²H²), 64× CPU speedup *with* AUC improvement, new stage
  [[feature-matching.reciprocal-matching]]) and
  [[coarse-to-fine-window-covering-matching_leroy2024]] (greedy
  full-res window set-cover; 4.6× Chamfer reduction on DTU zero-shot
  vs DUSt3R). Headline: **30 absolute AUC points over LoFTR+KBR on
  Map-free**; direct regression (no matching) achieves **94.1 VCRE
  AUC** — the strongest evidence that single-forward-pass Tier-3
  localization is competitive with matching-based pipelines when the
  pointmap is metric-scale. Gravitational centre for everything
  below.
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
- [FastVGGT (Shen 2025)](../papers/shen2025_fastvggt.md): training-free 4× speedup
  of VGGT via token merging with a VGGT-aware three-part partitioning (reference
  + salient + region-random). Headline: at 500+ frames FastVGGT actually
  *improves* pose accuracy over full-token VGGT — the cumulative-drift regime
  rewards aggressive 90% merging. Introduces stage
  [[feed-forward-sfm.token-compaction]]; idea
  [[vggt-token-merge-3-part-partition_shen2025]]. VGGT-compression family,
  token-count axis. Research-only license (inherits Meta VGGT license).
- [Faster VGGT with Block-Sparse Global Attention (Wang 2025)](../papers/wang2025_faster-vggt-block-sparse.md):
  training-free 4× speedup of VGGT's and π³'s global-attention layer via a
  kernel-agnostic binary block mask derived from pooled-Q̄K̄ᵀ similarity. Works on
  any block-sparse GPU kernel. Idea
  [[pooled-qk-block-sparse-global-attention_wang2025]] fills existing stage
  [[feed-forward-sfm.global-attention]] as a stage-swap alternative to the
  TTT-family fillers (LoGeR/ZipMap/TTT3R). VGGT-compression family, attention-
  sparsity axis. No code license — commercial-adoption-restricted.
- [QuantVGGT (Feng 2025)](../papers/feng2025_quantvggt.md): first PTQ for VGGT
  delivering W4A4 with ≥98% FP16 accuracy retention, 3.7× memory, 2.5× real-
  hardware speedup. Two co-required ideas:
  [[vggt-dual-smoothed-quantization_feng2025]] (Hadamard rotation + post-local
  channel smoothing to handle VGGT's data-independent special-token heavy tails)
  and [[frame-aware-ptq-calibration-sampling_feng2025]] (deep-layer noise
  filtering + frame-aware clustering for calibration stability). Introduces
  stages [[feed-forward-sfm.numerical-precision]] and
  [[feed-forward-sfm.ptq-calibration-sampling]]. VGGT-compression family,
  numerical-precision axis. **Commercial-license clean** (CC-BY-4.0 paper,
  Apache-2.0 code) — the only one of the three compression papers that is.

### Cross-cutting: matching and features
- [RoMa v2 (Edstedt 2025)](../papers/edstedt2025_roma-v2.md): dense matcher
  with DINOv3 backbone and custom CUDA refinement. Even in feed-forward SfM,
  correspondence quality still matters — RoMa v2 is the current matching SOTA.
- [Survey (Zhang 2025)](../papers/zhang2025_feed-forward-3d-survey.md):
  comprehensive taxonomy confirming the three-tier split above. Identifies
  pointmap prediction (DUSt3R-style) as the dominant feed-forward paradigm.
- [Survey (Wang 2026)](../papers/wang2026_feed-forward-3d-scene-modeling.md):
  second survey on the same field, orthogonal axis — organizes by
  **engineering problem** (feature enhancement / geometry awareness / model
  efficiency / augmentation / temporal-awareness) rather than by output
  representation. The new [[feed-forward-problem-axes]] concept page captures
  this as cross-cutting vocabulary. For this thread, the most concrete
  additions are the VGGT-efficiency-family search target (see Capability
  gaps), the §7.1 view-selection-bias critique, and the §6.4 observation
  that feed-forward SfM and feed-forward SLAM are converging into a single
  pipeline family (VGGSfM, Light3R-SfM, MASt3R-SLAM, SLAM3R, VGGT-SLAM,
  ARTDECO, MASt3R-Fusion, ViSTA-SLAM) — confirming Tier 3's direction.

## Emerging patterns

- **Test-time training** (LoGeR, ZipMap, TTT3R) is the scaling trick for feed-forward
  reconstruction: linear time, no quadratic KV cache, handles thousands of frames.
  Frontier question: does TTT need retraining (LoGeR, ZipMap) or can it be
  derived from the frozen model's own attention (TTT3R)? TTT3R's training-free
  closed-form learning rate is the cheapest answer yet.
- **VGGT-compression family as the parallel scaling trick.** Three orthogonal
  efficiency mechanisms populated 2026-04-21: token merging (FastVGGT, 4× at 1000
  frames, *improves* long-sequence pose via drift mitigation), block-sparse
  attention (Faster-VGGT, 4× on global attention, kernel-agnostic binary mask),
  W4A4 quantization (QuantVGGT, 3.7× memory × 2.5× speed, 98% accuracy retention).
  All three attack VGGT-class full-attention bottleneck from *different* resource
  axes — unlike TTT fillers which replace the attention function class, these
  preserve it and compute less. Theoretically stackable (Bet #029); head-to-head
  vs TTT-family untested (Bet #030).
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
- Will test-time-training replace attention for long-context 3D? Or will
  the VGGT-compression family (FastVGGT / QuantVGGT / SparseVGGT — flagged by
  [[wang2026_feed-forward-3d-scene-modeling]] §4.3 as the orthogonal
  efficiency branch) close the same deployment gap via full-attention
  compression?
- **Reconstruction-vs-generation spectrum** ([[wang2026_feed-forward-3d-scene-modeling]]
  §7.6): when should a feed-forward SfM output observed geometry only, and
  when should it hallucinate plausible structure for occluded / unseen
  regions? The current thread hypothesizes a strict reconstruction regime
  (match COLMAP accuracy) but a generation-augmented variant could materially
  help sparse-view + texture-poor cases — at the cost of metric fidelity.
  This question also lives in [[generative-3d-from-2d-priors]] and is the
  seam between the two threads.
- **Gauge-decoupled streaming** ([[wang2026_feed-forward-3d-scene-modeling]]
  §4.5.1: LongStream): keyframe-relative poses + cache-consistent training
  as a stability mechanism for very-long-sequence metric reconstruction — a
  concrete mechanism we haven't ingested; search target for the next round.

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

### Bet #026 — MASt3R metric pointmap as sequence-level conditioning for TTT3R / CUT3R state
status: proposed
combines: [[metric-scale-pointmap-loss_leroy2024]], [[ttt3r-closed-form-confidence-lr_chen2026]]
stage_target: feed-forward-sfm.recurrent-state-update
op_target: op:default (Tier 3)
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: TTT3R's closed-form confidence-guided learning rate drives a CUT3R-style recurrent pointmap state. The state is *up-to-scale* because CUT3R descends from DUSt3R. Injecting [[mast3r|MASt3R]]'s metric-scale pointmap output ([[metric-scale-pointmap-loss_leroy2024]]) as a per-step absolute-scale anchor gives the recurrent state a global reference frame without BA — closing the remaining sequence-level portion of the "calibration-grade outdoor" capability gap that Idea B only closes at the pair level. Anchoring per-step costs nothing inference-wise (MASt3R runs anyway) and zero retraining if applied as a Sim(3) alignment of the recurrent state each step.
expected_gain: calibration-grade multi-view ATE on outdoor ETH3D / ScanNet without BA tail; specifically, close the 2× ATE gap between MASt3R-SLAM (calibration-free) and DROID-SLAM (calibrated) observed in [[murai2025_mast3r-slam]]'s ETH3D-SLAM numbers.
risk: per-step Sim(3) alignment introduces scale-jitter if MASt3R's pair-wise metric predictions disagree on adjacent keyframes — known MASt3R failure mode (limitations ii in [[leroy2024_mast3r]]). Mitigation: low-pass-filter the scale estimate over the recurrent state's own confidence.
validating_experiment: modify [[chen2026_ttt3r|TTT3R]]'s closed-form LR to include a MASt3R-metric Sim(3) anchor per keyframe; rerun ETH3D-SLAM ATE/AUC vs the training-free baseline.
triggers: [ingest-of-idea:metric-scale-pointmap-loss_leroy2024]
created: 2026-04-21 · updated: 2026-04-21

### Bet #027 — Fast reciprocal-NN ([[fast-reciprocal-nn-matching_leroy2024]]) on RoMa v2 dense features
status: proposed
combines: [[fast-reciprocal-nn-matching_leroy2024]], [[roma-v2-dinov3-dense-matcher_edstedt2025]]
stage_target: feature-matching.reciprocal-matching
op_target: op:default (Tier 2 matcher frontend)
confidence: high
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: [[fast-reciprocal-nn-matching_leroy2024]] is matcher-agnostic — it operates on any pair of dense unit-norm descriptor maps. [[edstedt2025_roma-v2|RoMa v2]] currently uses its own custom CUDA refinement step, which is fast but not an explicit reciprocal filter. Replacing the CUDA refinement with MASt3R's `O(kWH)` cycle-finder at `k ≈ 3000` should (a) be GPU-friendly once ported from CPU FAISS, (b) retain RoMa's recall, and (c) apply the implicit outlier-filter cycle-consistency that RoMa v2 currently doesn't have. The paper showed the optimum at `k ≈ 3000` is robust enough to expect similar behavior on RoMa's descriptor space.
expected_gain: small AUC gain (+0.5–1 on MegaDepth / IMC) from cycle-consistency filtering, or failing that, a faster inference path for RoMa v2 at equal accuracy.
risk: RoMa v2's descriptors are higher-dim (>24) and trained with different supervision; the "fewer matches, higher AUC" monotonicity may not hold on that descriptor space.
validating_experiment: implement `k`-parameterized reciprocal matching on RoMa v2 dense output; sweep `k ∈ {768, 3000, 12000, 49000, all}` on MegaDepth / IMC / Map-free; compare against RoMa v2's native CUDA refinement.
triggers: [ingest-of-idea:fast-reciprocal-nn-matching_leroy2024]
created: 2026-04-21 · updated: 2026-04-21

### Bet #028 — MASt3R direct-regression pose as two-view bootstrap for MP-SfM
status: proposed
combines: [[metric-scale-pointmap-loss_leroy2024]], [[mono-depth-normal-constrained-incremental-sfm_pataki2025]]
stage_target: sfm.next-view-registration
op_target: [[pataki2025_mp-sfm|MP-SfM]] adopting thread's op:default (Tier 2)
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: MP-SfM's two-view bootstrap rescales mono-depth by the median ratio to already-triangulated points — scale-only alignment that breaks at 0%-overlap pairs ([[pataki2025_mp-sfm]] Eq. 1). Replacing the mono-depth + scale-fit with MASt3R's metric pointmap from Idea B gives bootstrap an *absolute-scale* pair of pointmaps that don't require alignment to a triangulated subgraph at all. Unlike Bet #014 (which uses MADPose's 3pt solver on unstructured mono-depth), this bet uses MASt3R's pointmap directly as the geometry seed. On ETH3D 0%-overlap (where mono-depth alone gives 34.9 AUC and MADPose adds +10), MASt3R direct regression should add more — its 94.1 AUC on Map-free under similar extreme-viewpoint conditions is the supporting data point.
expected_gain: +5–15 AUC@1° on ETH3D 0%-overlap over MP-SfM baseline; qualitatively: bootstrap that works even when zero keypoints match between pair members.
risk: MP-SfM's subsequent BA depends on consistent mono-depth across the full sequence — a MASt3R-only bootstrap followed by MP-SfM's mono-depth BA creates a modality mismatch. Mitigation: use MASt3R to seed scale, then switch to mono-depth for BA.
validating_experiment: rerun MP-SfM's ETH3D 0%-overlap benchmark with MASt3R direct-regression as the two-view bootstrap; compare AUC@1/5/10° and bootstrap success rate.
triggers: [ingest-of-idea:metric-scale-pointmap-loss_leroy2024]
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

### Bet #029 — Triple-orthogonal VGGT compression stack (token-merge × block-sparse × W4A4)
status: proposed
combines: [[vggt-token-merge-3-part-partition_shen2025]], [[pooled-qk-block-sparse-global-attention_wang2025]], [[vggt-dual-smoothed-quantization_feng2025]], [[frame-aware-ptq-calibration-sampling_feng2025]]
stage_target: feed-forward-sfm.token-compaction + feed-forward-sfm.global-attention + feed-forward-sfm.numerical-precision
op_target: op:default (Tier 3)
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: The three VGGT-compression papers each ablate only their own resource axis: FastVGGT reduces *token count* (4× at 1000 frames, training-free merge), Faster-VGGT reduces *attention density* (4× on global attention via binary block mask), QuantVGGT reduces *numerical precision* (3.7× memory × 2.5× speed at W4A4). None tests composition with the others. Because the three resource axes are genuinely independent, the gains should compose multiplicatively: ~16× global-attention speedup (token count × sparsity) × 2.5× numerical-precision speed × 3.7× memory reduction. On paper: ~40× end-to-end speedup, ~14× memory reduction vs FP16 dense VGGT at ≤2% accuracy loss.
expected_gain: on ScanNet-50 at 500-1000 frames, the triple stack matches FP16 VGGT's pose ATE within 10% at ~40× wall-clock speedup; establishes the VGGT-compression family's Pareto frontier vs TTT-family alternatives.
risk: (a) token merging changes activation distributions → QuantVGGT's calibration (trained on unmerged inputs) mis-quantizes the merged regime; (b) block-sparse mask quality degrades on compacted token streams because the pooled-similarity signal operates on fewer tokens; (c) compounded quantization + sparsification + merging may cross a nonlinear-regime threshold where accuracy collapses non-monotonically. Mitigation: (a) recalibrate QuantVGGT on merged-token streams (NFDS supports this — calibration pool just needs to include the merged regime); (b) recompute the sparsity-ratio `x` per token-count regime; (c) ablate staged adoption: merge-only → merge+sparse → merge+sparse+quant.
validating_experiment: sequential ablation on ScanNet-50 / 7-Scenes / NRGBD at 500/1000/2000 frames measuring {ATE, AUC@5, wall-clock, peak-VRAM, accuracy-retention-%} for {VGGT dense FP16, +merge, +merge+sparse, +merge+sparse+W8A8, +merge+sparse+W4A4}. Report the efficient frontier (Pareto curve of speed × accuracy).
triggers: [ingest-of-idea:vggt-token-merge-3-part-partition_shen2025, ingest-of-idea:pooled-qk-block-sparse-global-attention_wang2025, ingest-of-idea:vggt-dual-smoothed-quantization_feng2025]
created: 2026-04-21 · updated: 2026-04-21

### Bet #030 — VGGT-compression family vs TTT-scaling family on a common long-sequence benchmark
status: proposed
combines: [[vggt-token-merge-3-part-partition_shen2025]], [[pooled-qk-block-sparse-global-attention_wang2025]], [[vggt-dual-smoothed-quantization_feng2025]], [[ttt3r-closed-form-confidence-lr_chen2026]], [[large-chunk-ttt-fast-weights_jin2026]], [[ttt-mlp-kv-compression_elflein2026]], [[loger-hybrid-ttt-plus-swa_zhang2025]]
stage_target: feed-forward-sfm.global-attention (the shared bottleneck)
op_target: op:default (Tier 3)
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: The TTT-scaling family replaces VGGT's attention with a recurrent fast-weight state (linear time); the VGGT-compression family preserves the attention function class but computes less of it. Both target the same bottleneck. No paper compares branches directly. A head-to-head should reveal: (a) TTT's linear time wins at very long sequences (>2000 frames) where quadratic-but-compressed attention still loses; (b) VGGT-compression wins when the input distribution matches training (pure compression preserves function class), TTT wins under distribution shift (fast-weights adapt at inference); (c) combining them (W4A4 + TTT) may be unexpectedly synergistic — QuantVGGT's DSFQ is agnostic to attention mechanism.
expected_gain: first direct Pareto comparison across the two Tier-3 scaling branches; publication-quality insight regardless of which family wins; enables principled op selection (short-sequence / moderate-sequence / long-sequence) per capability-gap entry.
risk: benchmarks don't isolate *scaling behavior* well — most Tier-3 pose benchmarks saturate at <500 frames. The experiment may need a new long-sequence pose benchmark (gluing ScanNet-50 sequences, extending DTU MVS). This is partly already flagged in the view-selection-bias capability gap from [[wang2026_feed-forward-3d-scene-modeling]] §7.1.
validating_experiment: run {dense VGGT, TTT3R, ZipMap, VGG-T3, LoGeR, FastVGGT, Faster-VGGT, QuantVGGT, triple stack from Bet #029} on ScanNet-50 / ScanNet-gluing-to-2000-frames / ETH3D-gluing / 7-Scenes at 500 / 1000 / 2000 frames. Report ATE, AUC@5, wall-clock, peak-VRAM. Produce the efficient frontier per sequence length.
triggers: [ingest-of-idea:vggt-token-merge-3-part-partition_shen2025, ingest-of-idea:pooled-qk-block-sparse-global-attention_wang2025, ingest-of-idea:vggt-dual-smoothed-quantization_feng2025]
created: 2026-04-21 · updated: 2026-04-21

