---
title: Foundation Features for Geometry
type: thread
tags: [foundation-model, dinov2, dinov3, feature-matching, sfm, frozen-backbone]
created: 2026-04-15
updated: 2026-04-22
sources: [wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/edstedt2025_roma-v2.md, wiki/papers/jang2025_pow3r.md, wiki/papers/zhang2025_feed-forward-3d-survey.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/ranzinger2026_c-radiov4.md, wiki/papers/radford2021_clip.md, wiki/papers/kirillov2023_sam.md, wiki/papers/leroy2024_mast3r.md]
operating_points: [op:default]
status: draft
---

## Goal

Produce the **feature backbone + task head** pattern that serves as the frontend for geometry problems (matching, pose, mono/multi-view depth, feed-forward pointmaps). Better = higher matching recall / lower pose AUC / lower Chamfer on reconstruction benchmarks at equal or lower head training cost, while preserving robustness under the failure modes classical hand-crafted descriptors struggle with (textureless surfaces, illumination/weather changes, repetitive structure).

## Goal contract (optional, structured)

```yaml
metric: [pose-AUC@5deg, dense-match-AEPE, depth-AbsRel, backbone-training-cost]
target_regime: [frozen-backbone, head-trainable-hours-to-days-on-1-to-8-GPUs]
constraints: [backbone-must-stay-frozen, head-inference-<=100ms, graceful-on-low-texture]
required_capabilities: [patch-level-spatial-consistency, cross-domain-generalization, multi-task-reuse]
```

## SOTA pipelines

### op:default

Stage-by-stage, for a generic geometry task ("given images, extract features used by a downstream geometric head"):

1. **Image tokenization / patch embedding** — ViT patch embed, 14-px patches. Component: DINOv3 ViT-L/14 (default) or ViT-g/14 when compute allows. Paper: [[simeoni2025_dinov3]]. Gain over DINOv2: Gram anchoring improves patch-level consistency at high resolution; measured on dense-matching and segmentation downstream tasks. Failure mode: 14-px patches lose fine structure; silent failure on low-texture regions.
2. **Frozen backbone forward pass** — extract per-patch features at the chosen layer. Component: DINOv3 (default) for pure geometry; [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] (paper: [[ranzinger2026_c-radiov4]]) when the task *also* needs semantic alignment (i.e., multi-task downstream). Predecessor: [[heinrich2025_radiov25|RADIOv2.5]], retained as fallback under NSCL research license. C-RADIOv4 is best-in-family on Probe3D NAVI (63.44) and SPair (60.57) — the geometry-relevant Probe3D metrics. Failure mode: C-RADIOv4 / DINOv3 features are *not* text-aligned without the SigLIP2 adaptor — projects that need language queries must use the adaptor output rather than the raw student features.
3. **Task head** — small transformer / regression head on frozen features. Examples: RoMa v2 dense-match head, Pow3R pointmap head, DUSt3R/MASt3R/VGGT pointmap heads, Metric3Dv2 depth head. Training cost: hours to 1–2 days on 1–8 GPUs.
4. **Geometric refinement (optional)** — feed-forward output fed into classical BA / factor graph. See [[feed-forward-structure-from-motion]] Tier 2 for the hybrid pattern.

## Pipeline lineage

- 2021 · backbone: SIFT / learned-from-scratch descriptors → CLIP image-features (for semantics only; not geometry) · driver: [[radford2021_clip]] — *excluded* from geometry pipeline; CLIP is text-aligned but spatially weak.
- 2023 · backbone: hand-crafted / task-trained → DINOv2 · driver: [[oquab2023_dinov2]]. Gain: first self-supervised ViT that beat task-specific descriptors on multiple downstream geometry heads without fine-tuning.
- 2025 · backbone: DINOv2 → DINOv3 · driver: [[simeoni2025_dinov3]]. Gram anchoring for dense patch consistency.
- 2025 · multi-task frontend: single-teacher DINOv2 → RADIOv2.5 distillation of DINOv2 + CLIP + SigLIP + SAM · driver: [[heinrich2025_radiov25]]. Used where geometry + semantics share a frontend.
- 2026-04-22 · op:default · multi-task frontend filler-swap: RADIOv2.5 → [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] (SigLIP2 + DINOv3 + SAM3 teacher set, commercial license, Probe3D NAVI/SPair best-in-family at +2.5 / +4.3 over RADIOv2.5-H) · driver: [[ranzinger2026_c-radiov4]]. RADIOv2.5 remains fallback for direct comparability with published baselines.
- 2025 · dense-matching head: LoFTR / SuperGlue → RoMa v2 on DINOv3 · driver: [[edstedt2025_roma-v2]].
- 2026 · repurposing the frozen backbone as training-dynamics source: TTT3R uses DINO attention alignment as a closed-form learning rate · driver: [[chen2026_ttt3r]]. This is a *new stage* in the taxonomy — the frozen features are not just inputs.

## Candidate components / not yet integrated

- **SigLIP** (Zhai 2023) — pairwise sigmoid contrastive; drop-in for CLIP in multi-task frontends. Not yet a stub; referenced by multiple papers. Likely wins for the semantic-alignment lane when that lane is needed. Blocked on: stub creation; no systematic geometry-downstream comparison yet.
- **EVA-CLIP** — MIM + CLIP pretraining. Same lane as SigLIP; secondary candidate.
- **3D-native self-supervision** (video-aware DINO, 4D pretraining) — speculative. Status: no paper yet in the wiki; flagged as the "next backbone generation" bet.

## Open questions & synthesis bets

### Bet #011 — DINOv3 features + classical geometric-consistency rejection head
status: proposed
combines: [[simeoni2025_dinov3]], [[edstedt2025_roma-v2]]
stage_target: feature-matching.rejection
op_target: op:default
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: DINO-based geometry fails silently with plausible-but-wrong outputs where SIFT failed loudly. Combining DINOv3 features with a classical geometric-cycle-consistency check as a rejection head (and using the inconsistency as a self-supervised fine-tuning signal) bridges the backbone-confidence gap that TTT3R opens.
expected_gain: 3–8% false-match-rate reduction on low-texture benchmarks; small pose-AUC gain but large qualitative win on silent-failure cases.
risk: Geometric consistency cost dominates inference; needs a lightweight variant to stay deployable. Rejection thresholds are dataset-dependent.
validating_experiment: Add cycle-consistency rejection head on top of RoMa v2; ablate vs. RoMa v2 baseline on textureless scenes (MegaDepth-hard, custom indoor textureless).
triggers: [ingest-of-idea:lightweight-geometric-cycle-consistency]
created: 2026-04-15 · updated: 2026-04-18

### Bet #012 — TTT3R's closed-form learning rate on RoMa v2's dense matching head
status: proposed
combines: [[chen2026_ttt3r]], [[edstedt2025_roma-v2]]
stage_target: feature-matching.task-head
op_target: op:default
confidence: med
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: TTT3R derives a closed-form confidence-guided per-token learning rate from frozen backbone attention — no retraining needed. Extending the same derivation to RoMa v2's dense matching head gives per-match adaptive learning rates without architectural change.
expected_gain: Smoother convergence on hard scenes; +0.5–1.0 pose-AUC@5 on MegaDepth / ScanNet.
risk: RoMa v2 already has a CUDA-optimized inner loop; per-match learning-rate modulation may be hard to integrate without giving up speed.
validating_experiment: Implement TTT3R-style confidence-guided LR on RoMa v2's head; ablate on MegaDepth + ScanNet.
triggers: [ingest-of-idea:closed-form-adaptive-lr-for-matching]
created: 2026-04-15 · updated: 2026-04-18

## Capability gaps

- **Silent-failure rejection signal** — DINO geometry fails plausibly-wrong where SIFT failed loudly. Would unlock: joint DINO+classical-consistency residual bets. Search target: papers that fuse foundation features with geometric-cycle-consistency rejection.
- **3D-native self-supervised backbone** (video-aware / 4D DINO) — would unlock the next generation of geometry heads. Search target: 2026 SSL papers training on multi-view / video data natively.
- **Calibration-grade feed-forward poses without classical BA** — would obsolete the hybrid tier. **Partially closed at the pair-pose level** by [[leroy2024_mast3r|MASt3R]] (ingested 2026-04-21): [[metric-scale-pointmap-loss_leroy2024]] gives 94.1 VCRE AUC / 0.42m median translation on Map-free from a single forward pass. But MASt3R's backbone is DUSt3R/CroCo-pretrained, *not* DINOv3 — so the win is evidence that **3D-grounded pretraining can substitute for a stronger 2D frozen backbone** for this thread's goal. Open question: could a DINOv3-backboned equivalent of Idea B close the same gap? Captured downstream in Bet #028 synthesis direction from [[feed-forward-structure-from-motion]]. Search target: 2026 papers attaching a MASt3R-style metric pointmap head to a DINOv3 (rather than CroCo) frozen backbone.

## Contradictions & tensions

- Does DINOv3's Gram anchoring benefit geometric heads in proportion to its segmentation gains? No systematic ablation yet. Several matching papers report 0.5–2% pose-AUC gains, but some geometry-head ablations show no improvement over DINOv2 at equivalent compute — unresolved.

## Working hypothesis

Between ~2023 and 2026, **frozen self-supervised ViT features** ([[dinov2|DINOv2]], DINOv3) have displaced hand-crafted + task-trained descriptors as the feature backbone for geometric computer vision — SfM, matching, monocular depth, feed-forward reconstruction. The "foundation backbone + task head" pattern is now standard. SIFT-era assumptions (rotation/scale-invariant local descriptors, carefully engineered frontend) are losing ground to "just freeze DINOv2 and regress."

## Evidence

### Feed-forward 3D reconstruction
- [[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]], [[jang2025_pow3r|Pow3R]], [[CUT3R]] — all use DINOv2 as a frozen backbone and attach a pointmap / pose / matching head on top. See [[feed-forward-structure-from-motion]] for the sister thread focused on the pipeline side.
- [TTT3R (Chen 2026)](../papers/chen2026_ttt3r.md) is a noteworthy extension of the frozen-backbone pattern: it repurposes the **alignment confidence between frozen features** (state queries vs. observation keys from the DINO/CroCo-tokenized ViT) as a closed-form per-token learning rate for a test-time state update. The frozen backbone isn't just a feature extractor — its attention signal *is* the meta-learner. Evidence that the "composable frozen backbone" pattern extends beyond features into training dynamics.

### Dense matching
- [[edstedt2025_roma-v2|RoMa v2]] — frozen DINOv3 features drive dense correspondence. Beats hand-crafted SIFT, learned SuperGlue/LoFTR on a wide benchmark sweep. The paper explicitly argues DINOv3 features make specialized matching losses largely unnecessary.
- [[leroy2024_mast3r|MASt3R]] (contrast) — rather than freezing a large 2D backbone and training a matching head on top, MASt3R *jointly trains* a dense-descriptor head alongside a 3D pointmap head on a fine-tuned DUSt3R/CroCo ViT ([[dust3r-matching-head_leroy2024]]). The 3D supervision substitutes for DINOv3's self-supervised visual prior: matching accuracy (30 absolute AUC points over LoFTR+KBR on Map-free) comes from 3D grounding, not frozen-backbone quality. This is evidence that "foundation features" need not mean "frozen DINOv3" for geometry — a 3D-task-pretrained encoder is a parallel design point worth tracking.
- **Cross-cutting**: [[fast-reciprocal-nn-matching_leroy2024]] is a matcher-agnostic dense-reciprocal algorithm (new stage [[feature-matching.reciprocal-matching]]). It applies to RoMa v2 output just as well as MASt3R output — a drop-in efficiency atom for anything in this thread's task-head section. Captured as Bet #027 of [[feed-forward-structure-from-motion]].

### Monocular depth & geometric priors
- Depth Anything / DINOv2-based depth heads follow the same pattern: freeze the backbone, train a depth decoder on mixed-domain data.
- See [[mono-depth-estimation]] for the downstream usage of depth priors in SfM.

### Unified vision backbones
- [[heinrich2025_radiov25|RADIOv2.5]] distills CLIP + SigLIP + DINOv2 + SAM into one encoder — evidence that the "composable frozen backbone" role is being formalized across the field.

## Why it works

- **Robustness**: DINO features are trained on 142M–1B+ images; they generalize across domains where SIFT collapses (textureless, strong illumination changes, weather, seasonal variation).
- **Dense + semantic + geometric**: patch-level SSL produces features that are simultaneously spatially coherent (DINO self-similarity) and semantically meaningful — a property SIFT lacked.
- **Training cost**: the head is small (a few layers of attention or regression); training for a new task is hours on 1–8 GPUs, vs. weeks for a from-scratch geometric model.

## Open questions / tensions

- **Calibration-grade poses**: feed-forward methods still lag classical SfM ([[colmap|COLMAP]], [[pan2024_glomap|GLOMAP]]) on metric accuracy for calibration-critical workflows. Is the gap closable with larger backbones, or is refinement/BA still load-bearing?
- **Failure modes**: low-texture and repetitive structure — where SIFT at least failed loudly. DINO-based methods tend to fail *silently* with plausible-but-wrong outputs.
- **DINOv3's Gram anchoring**: do downstream geometric heads benefit proportionally, or do they only care about coarse features? No systematic ablation yet.
- **Computational footprint**: ViT-g/14 is ~1.1B params; smaller distilled variants are preferred for embedded use but lose some quality.

## Relation to other threads
- [[feed-forward-structure-from-motion]] — pipeline-side sibling focused on what replaces the classical SfM loop.
- [[gpu-native-sfm]] — accelerated *classical* SfM, orthogonal axis (speed, not representation).
- [[lifting-foundation-models-to-3d]] — the same DINO/CLIP/SAM features, but lifted into 3D representations for segmentation and language grounding.

## Outstanding hypotheses
- *(Provisional)* The next major gain comes from **joint self-supervised pretraining across 2D images and 3D geometry** (video-aware DINO, 4D pretraining) rather than scaling DINOv3 further. Watch for evidence in 2026.
- *(Pass B 2026-04-22)* [[shift-equivariant-distillation-loss_ranzinger2026]] identifies a named failure mode — fixed-pattern positional noise inherited from teacher register tokens and ViTDet window borders. Published matching heads (RoMa v2, MASt3R, DUSt3R descriptor head) train on frozen DINO/CroCo features; none reports whether these artifacts degrade matching on repeated-texture or low-texture surfaces. Hypothesis: applying shift-equivariance at matching-head training time (random patch-aligned shifts of the query and target independently) would reduce false-positive matches on register-token grid patterns without changing the backbone. Low-cost test. Flagged as future bet candidate.

