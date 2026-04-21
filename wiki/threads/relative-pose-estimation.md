---
title: Relative Pose Estimation (two-view, depth-aware)
type: thread
tags: [pose-estimation, relative-pose, two-view, minimal-solver, ransac, msac, mono-depth, epipolar]
created: 2026-04-21
updated: 2026-04-21
sources: [papers/yu2025_madpose.md, papers/pataki2025_mp-sfm.md, papers/leroy2024_mast3r.md]
operating_points: [op:default]
status: stable
---

## Goal

Estimate the relative pose `(R, t)` between two images as accurately as possible given modern correspondence + monocular-depth priors — in calibrated, shared-focal, and fully-uncalibrated settings. Two-view pose is a distinct problem from multi-view SfM (this thread's sibling [[feed-forward-structure-from-motion]]): the benchmark suite is fixed (ScanNet-1500 indoor, MegaDepth-1500 outdoor), the solvers are minimal (3 to 7 correspondences), and the robust estimator (LO-RANSAC / LO-MSAC) is the frame every paper optimizes. The thread tracks how modern learned priors — matcher (SP+LG, RoMa, MASt3R) and depth (DA, Marigold, MoGe, feed-forward pointmap) — can be fused with classical minimal solvers to beat point-only baselines, especially in regimes where depth priors are reliable but epipolar geometry is weak (short baseline, narrow field of view, texture-poor).

> **Thread created 2026-04-21 by ingest of [[yu2025_madpose|MADPose]]** (per §3.3 goal-first workflow). Goal + contract + OPs drafted from seed papers.

## Goal contract (optional, structured)

```yaml
metric: [pose-AUC@5deg, pose-AUC@10deg, pose-AUC@20deg, median-rot-err-deg, median-trans-err-deg, median-focal-err-percent]
target_regime: [two-view, calibrated | shared-focal | two-focal-uncalibrated, static-scene]
constraints: [RANSAC-compatible-solver, graceful-degradation-when-depth-prior-unreliable]
required_capabilities: [minimal-solver, robust-scoring, local-optimization]
# open: whether to add an SfM-compatible "pair-pose as SfM init" sub-regime as a separate OP later
```

## SOTA pipelines

### op:default (two-view relative pose, any calibration)

The SOTA pipeline for depth-aware two-view pose, using MADPose as the instantiation:

- **n1** · stage: [[feature-matching.task-head]] · filler: _matcher-of-choice_ (SP+LG, RoMa v2 on DINOv3 from [[roma-v2-dinov3-dense-matcher_edstedt2025]], or MASt3R via [[dust3r-matching-head_leroy2024]] — 3D-grounded dual-head matching) · source: various; MASt3R = [Leroy et al. 2024](../papers/leroy2024_mast3r.md) · gain over prior: dense / semi-dense matchers provide 10× more correspondences than SIFT+NN; 3D-grounded MASt3R adds extreme-viewpoint robustness · upstream: [raw-image-pair] · downstream: [n3] · edge types: `in: image-pair; out: pixel-correspondences`.
- **n2** · stage: [[mono-depth-estimation]] (implicit stage; see [[mono-depth-estimation]] thread) · filler: off-the-shelf MDE (MoGe best calibrated, DA-met best shared-focal/two-focal per [[yu2025_madpose]]) · source: [[wang2025_moge]] / Depth-Anything v2 · gain: affine-invariant or metric depth per view at 3–160 ms / image · upstream: [raw-image-pair] · downstream: [n3] · edge types: `in: image; out: per-view-depth-map`.
- **n3** · stage: [[pose-estimation.hybrid-robust-estimator]] · filler: [[hybrid-lo-msac-dual-modality-estimator_yu2025]] · source: [Yu et al. 2025](../papers/yu2025_madpose.md) · gain over prior: ScanNet-1500 shared-focal + MASt3R + DA-met **56.99 AUC@10° vs 30.27 PoseLib-6pt baseline (+27)**; MegaDepth-1500 two-focal +16 AUC@10° · upstream: [n1, n2] · downstream: [n4] · edge types: `in: correspondences, depth; out: R, t, α, β_1, β_2, [f_1, f_2], inliers`. **Bundle node**: filler has `co_requires: [affine-corrected-minimal-relative-pose-solvers_yu2025, depth-induced-reprojection-scoring_yu2025]` — those two ideas must also appear as fillers below (bundle travels together per §6.14).
- **n4** · stage: [[pose-estimation.relative-pose-solver]] · filler: [[affine-corrected-minimal-relative-pose-solvers_yu2025]] (bundled with n3) · source: [Yu et al. 2025](../papers/yu2025_madpose.md) · gain: +15 AUC@20° from shift modeling vs scale-only (Table 6) · upstream: [n1, n2] · downstream: [n3] · edge types: `in: minimal-sample; out: pose-hypotheses-with-affine-corrections`.
- **n5** · stage: [[pose-estimation.robust-estimator-scoring]] · filler: [[depth-induced-reprojection-scoring_yu2025]] (bundled with n3) · source: [Yu et al. 2025](../papers/yu2025_madpose.md) · gain: largest single ablation contributor in Table 7 (−8 AUC@10° when dropped) · upstream: [n3, n4] · downstream: [n3] · edge types: `in: pose-hypothesis, correspondence, corrections; out: per-correspondence-score`.

The three MADPose ideas (n3, n4, n5) form a **bundle node** — they must all be present in any adopting pipeline. The hybrid estimator is the composite filler; the solver and scoring ideas are its constituent sub-fillers.

## Pipeline lineage

- 2023 · op:default · filler-swap · n3: _N/A → PoseLib-5pt/6pt/7pt RANSAC_ · driver: [[colmap-incremental-sfm_schonberger2016|COLMAP-era baseline]] · gain: reference point-only baseline (ScanNet-1500 SP+LG calibrated AUC@10° ≈ 39).
- 2024 · op:default · filler-swap · n3: _PoseLib-5pt → Barath 2022 2pt+D_ · driver: Barath & Sweeney 2022 ([4] in [[yu2025_madpose]]) · gain: minor AUC@10° lift (−5 to +0 depending on domain) · outcome: scale-only modeling rank-deficient; sometimes regresses.
- 2024 · op:default · filler-swap · n3: _PoseLib-6pt → Ding 2024 3p3d/4p4d_ · driver: Ding et al. 2024 ([17] in [[yu2025_madpose]]) · gain: shared-focal/two-focal scale-only depth-aware solvers; inconsistent outdoor gains.
- 2026-04-21 · op:default · topology-change · scope: stage-split · nodes removed: `n3 (old monolithic RANSAC)` · nodes introduced: `n3 (hybrid estimator), n4 (affine-correcting solver), n5 (depth-induced scoring)` · driver: [[yu2025_madpose|MADPose]] · rationale: the MADPose hybrid estimator is not a single-node filler — its value comes from the composition of solver + scoring + selection-based wrapper, so the SOTA pipeline reflects the three-node bundle. This is a `stage-split` in spirit (the old monolithic `pose-estimation.*` stage had no explicit sub-structure; we now have three) captured as a topology-change.

<details>
<summary>Prior (2024) SOTA</summary>

Prior to MADPose, the community SOTA was **PoseLib-5pt/6pt/7pt + Sampson MSAC** for each calibration regime, with depth priors unused at pose-estimation time (depth was reserved for downstream SfM or 3DGS). Barath 2022 and Ding 2024 explored scale-only depth augmentation but did not consistently beat point-only baselines.

</details>

## Candidate components / not yet integrated

- **MASt3R direct-regression pose** (via [[metric-scale-pointmap-loss_leroy2024]]): not on the same benchmark as MADPose's Table 5 — the MASt3R Map-free result (94.1 VCRE AUC, 0.42m median translation from direct PnP on the predicted metric pointmap, no matching, no RANSAC) and the MegaDepth two-focal result (36.80 AUC@10° in Table 5 of [[yu2025_madpose]], used as a _matcher_ into MADPose) are measuring different things. The right reading: **MASt3R's metric pointmap wins on calibrated / known-viewpoint-class benchmarks (Map-free) where direct regression from ẑ is well-defined, but loses on uncalibrated / general two-view benchmarks where intrinsics alignment breaks the metric assumption** — MADPose's robust estimator dominates there. Candidate integration: use MASt3R direct-regression when the task has known-scale priors (indoor AR, map-free re-localization), swap to MADPose otherwise. Runtime argument (single forward pass vs RANSAC) is secondary.
- **Fast reciprocal-NN matcher** ([[fast-reciprocal-nn-matching_leroy2024]]): drop-in speedup for n1 regardless of matcher choice. 64× CPU speedup with AUC improvement on the Map-free benchmark; matcher-agnostic. Should be part of every n1 configuration.
- **Feed-forward pointmap pose** (DUSt3R / MASt3R two-view output used directly): Table 5 of [[yu2025_madpose]] shows MASt3R alone gets 36.80 AUC@10° on MegaDepth-1500 two-focal, beaten by MADPose (+16) using the same MASt3R matches and DA-met depth — meaning the MASt3R predicted pose is weaker than MADPose's classical solver with MASt3R's matches + DA-met's depth. This is a candidate because it might still win on *runtime* (single forward pass, no RANSAC).
- **Barath 2022 2pt+D, Ding 2024 3p3d/4p4d** (scale-only predecessors): parked in lineage — superseded on accuracy, retained as historical points.
- **PoseLib-5pt/6pt/7pt as lone fillers**: lineage entries; the point-based solver still lives inside the hybrid estimator as the fallback modality.

## Open questions & synthesis bets

### Bet #014 — MADPose solver as two-view init inside [[pataki2025_mp-sfm|MP-SfM]]
- status: proposed
- combines: [[[affine-corrected-minimal-relative-pose-solvers_yu2025]], [[mono-depth-normal-constrained-incremental-sfm_pataki2025]]]
- stage_target: `sfm.next-view-registration` (specifically the two-view bootstrap sub-step of MP-SfM §3.1)
- op_target: [[pataki2025_mp-sfm|MP-SfM]] adopting thread's op:default ([[feed-forward-structure-from-motion]] Tier 2)
- confidence: high
- magnitude: incremental
- cost: days
- breakage_risk: low
- hypothesis: MP-SfM's two-view bootstrap (Eq. 1) rescales mono-depth by the median ratio to already-triangulated points — a `α`-only alignment. Replacing the rescaling with MADPose's calibrated 3pt solver jointly solves `(α, β_1, β_2, R, t)`, upgrading the bootstrap from scale-only to affine alignment. On MP-SfM's 0%-overlap ETH3D subset, where two-view init is the hardest sub-problem, this should produce a measurable gain.
- expected_gain: +5–10 AUC@1° on ETH3D 0%-overlap (MP-SfM reports 34.9 vs GLOMAP 8.4; MADPose's +15 shift-modeling gain at the pair level should partially propagate).
- risk: MP-SfM's later BA steps may wash out the pair-pose improvement if the shift estimate doesn't propagate across pairs — this is the known limitation noted in `affine-corrected-minimal-relative-pose-solvers_yu2025`'s open questions. Mitigation: re-solve `β` per pair and fix only on the longest-overlap shared pair.
- validating_experiment: re-run MP-SfM's ETH3D 0%-overlap benchmark with the calibrated 3pt solver as two-view init; compare AUC@1°/5° and two-view-init success rate.
- triggers: [ingest-of-idea:affine-corrected-minimal-relative-pose-solvers_yu2025]
- created: 2026-04-21 · updated: 2026-04-21

### Bet #015 — Depth-induced reprojection as a third model in COLMAP multi-model geometric verification
- status: proposed
- combines: [[[depth-induced-reprojection-scoring_yu2025]], [[colmap-incremental-sfm_schonberger2016]]]
- stage_target: `sfm.geometric-verification`
- op_target: [[gpu-native-sfm]] op:general-purpose (InstantSfM) — the Tier-1 thread where multi-model RANSAC already runs
- confidence: med
- magnitude: incremental
- cost: days
- breakage_risk: low
- hypothesis: COLMAP's geometric verification runs H + F multi-model RANSAC to catch panorama/planar degeneracies. Adding depth-induced reprojection as a third model catches scenes where epipolar passes but metric depths disagree — a class of degeneracy H + F cannot see. Especially useful for texture-poor / repetitive scenes where depth prior is the most discriminative signal.
- expected_gain: higher verification pass rate on hard pairs (0%-overlap-style, texture-poor) without hurting easy pairs. No headline metric is directly published for this ablation.
- risk: when depth prior is wrong, the third model could reject valid pairs. Mitigation: use as a soft vote (`2-of-3` must pass), not a hard filter.
- validating_experiment: ablate on ETH3D texture-poor subset; compare pair verification rate + downstream SfM registration success.
- triggers: [ingest-of-idea:depth-induced-reprojection-scoring_yu2025]
- created: 2026-04-21 · updated: 2026-04-21

### Bet #016 — MADPose pair-pose frontend for InstantSfM global positioning
- status: proposed
- combines: [[[hybrid-lo-msac-dual-modality-estimator_yu2025]], [[glomap-joint-global-positioning_pan2024]], [[depth-constrained-jacobian_zhong2026]]]
- stage_target: pair-pose frontend feeding [[sfm.global-positioning]]
- op_target: [[gpu-native-sfm]] op:general-purpose
- confidence: med
- magnitude: substantial (if it lands)
- cost: weeks
- breakage_risk: med
- hypothesis: InstantSfM inherits its pair-pose quality from COLMAP's 5pt RANSAC + view-graph construction. Swapping to MADPose's hybrid estimator as the pair-pose step (producing `R, t, α, β_1, β_2`) feeds the global-positioning stage with better initialization and per-pair shift estimates that could constrain the global depth frame. Especially impactful on outdoor / wide-baseline where MADPose's gains are largest.
- expected_gain: +2–5 AUC@5° on MipNeRF360 NVS (InstantSfM's headline benchmark), potentially larger on outdoor ScanNet++/ETH3D-outdoor subsets.
- risk: MADPose's per-pair `β` estimates disagree across pairs sharing a view — the propagation problem. Resolving this is the main technical risk.
- validating_experiment: replace InstantSfM's pairwise-pose step with MADPose; benchmark on MipNeRF360 + one outdoor scene set.
- triggers: [ingest-of-idea:hybrid-lo-msac-dual-modality-estimator_yu2025]
- created: 2026-04-21 · updated: 2026-04-21

## Capability gaps

- **Shift-propagation across a sequence** — MADPose's per-pair `β_1, β_2` estimates don't compose into a globally-consistent per-image shift. Would unlock: Bet #014, Bet #016. Search target: papers proposing global mono-depth-affine consistency (possibly [[pataki2025_mp-sfm|MP-SfM]]'s Eq. 1 extended with shift, or unpublished follow-ups).
- **Degeneracy theory for depth-aware solvers** — current graceful-degradation claims are empirical. Would unlock: confidence bounds on Bet #016 production deployment. Search target: coplanarity / degeneracy analysis for 3-point depth-aware solvers.
- **Matcher robustness on texture-poor regimes** — MADPose tests SP+LG / RoMa / MASt3R, all strong. How does the hybrid estimator behave on texture-poor pairs where detector-free matchers ([[he2023_detector-free-sfm|DetectorFreeSfM]]'s LoFTR-class) produce many noisy correspondences? Would unlock: cross-thread synergy with [[he2023_detector-free-sfm]]'s front-end. Search target: ablations of hybrid estimators on IMC Texture-Poor subset.

## Contradictions & tensions

- **Shift modeling vs. metric depth**: conventional wisdom says metric-trained MDE (e.g., DA-met) needs only scale correction, not shift. [[yu2025_madpose]] Fig. 6 contradicts this — fitted `β` > 10% of median depth is common even for metric-trained models. The paper interprets this as per-image bias in the metric model; an alternative interpretation is that GT depth (SfM) is itself scale-shifted. Both explanations fit the data; the wiki lists the empirical finding as load-bearing (adoption decisions in Bets #014-016 assume shift is real).

## Sources

- [[yu2025_madpose|MADPose]] (primary driver)
- [[pataki2025_mp-sfm|MP-SfM]] (two-view bootstrap is a downstream integration target)
