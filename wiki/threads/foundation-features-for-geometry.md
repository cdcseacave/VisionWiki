---
title: Foundation Features for Geometry
type: thread
tags: [foundation-model, dinov2, dinov3, feature-matching, sfm, frozen-backbone]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/edstedt2025_roma-v2.md, wiki/papers/jang2025_pow3r.md, wiki/papers/zhang2025_feed-forward-3d-survey.md, wiki/papers/heinrich2025_radiov25.md, wiki/papers/radford2021_clip.md, wiki/papers/kirillov2023_sam.md]
status: draft
---

## Goal & success criteria

Produce the **feature backbone + task head** pattern that serves as the frontend for geometry problems (matching, pose, mono/multi-view depth, feed-forward pointmaps). "Better" = higher matching recall / lower pose AUC error / lower Chamfer on reconstruction benchmarks *at equal or lower head training cost*, while preserving robustness under the failure modes classical hand-crafted descriptors struggle with (textureless surfaces, illumination/weather changes, repetitive structure).

## Current SOTA pipeline (as of 2026-04-15)

Stage-by-stage, for a generic geometry task ("given images, extract features used by a downstream geometric head"):

1. **Image tokenization / patch embedding** — ViT patch embed, 14-px patches. Component: DINOv3 ViT-L/14 (default) or ViT-g/14 when compute allows. Paper: [[simeoni2025_dinov3]]. Gain over DINOv2: Gram anchoring improves patch-level consistency at high resolution; measured on dense-matching and segmentation downstream tasks. Failure mode: 14-px patches lose fine structure; silent failure on low-texture regions.
2. **Frozen backbone forward pass** — extract per-patch features at the chosen layer. Component: DINOv3 (default) for pure geometry; [[heinrich2025_radiov25]] RADIOv2.5 when the task *also* needs semantic alignment (i.e., multi-task downstream). Failure mode: DINOv3 features are *not* text-aligned — projects that need language queries must route through a different backbone.
3. **Task head** — small transformer / regression head on frozen features. Examples: RoMa v2 dense-match head, Pow3R pointmap head, DUSt3R/MASt3R/VGGT pointmap heads, Metric3Dv2 depth head. Training cost: hours to 1–2 days on 1–8 GPUs.
4. **Geometric refinement (optional)** — feed-forward output fed into classical BA / factor graph. See [[feed-forward-structure-from-motion]] Tier 2 for the hybrid pattern.

## Pipeline lineage

- 2021 · backbone: SIFT / learned-from-scratch descriptors → CLIP image-features (for semantics only; not geometry) · driver: [[radford2021_clip]] — *excluded* from geometry pipeline; CLIP is text-aligned but spatially weak.
- 2023 · backbone: hand-crafted / task-trained → DINOv2 · driver: [[oquab2023_dinov2]]. Gain: first self-supervised ViT that beat task-specific descriptors on multiple downstream geometry heads without fine-tuning.
- 2025 · backbone: DINOv2 → DINOv3 · driver: [[simeoni2025_dinov3]]. Gram anchoring for dense patch consistency.
- 2025 · multi-task frontend: single-teacher DINOv2 → RADIOv2.5 distillation of DINOv2 + CLIP + SigLIP + SAM · driver: [[heinrich2025_radiov25]]. Used where geometry + semantics share a frontend.
- 2025 · dense-matching head: LoFTR / SuperGlue → RoMa v2 on DINOv3 · driver: [[edstedt2025_roma-v2]].
- 2026 · repurposing the frozen backbone as training-dynamics source: TTT3R uses DINO attention alignment as a closed-form learning rate · driver: [[chen2026_ttt3r]]. This is a *new stage* in the taxonomy — the frozen features are not just inputs.

## Candidate components / not yet integrated

- **SigLIP** (Zhai 2023) — pairwise sigmoid contrastive; drop-in for CLIP in multi-task frontends. Not yet a stub; referenced by multiple papers. Likely wins for the semantic-alignment lane when that lane is needed. Blocked on: stub creation; no systematic geometry-downstream comparison yet.
- **EVA-CLIP** — MIM + CLIP pretraining. Same lane as SigLIP; secondary candidate.
- **3D-native self-supervision** (video-aware DINO, 4D pretraining) — speculative. Status: no paper yet in the wiki; flagged as the "next backbone generation" bet.

## Open questions & synthesis bets

- Calibration-grade poses: feed-forward methods still lag [[colmap]] / GLOMAP on metric accuracy. **Open**: is the gap closable by scaling the frozen backbone, or is classical BA refinement always load-bearing? Current lineage evidence says the latter — every Tier 2 paper in [[feed-forward-structure-from-motion]] keeps BA at the end.
- **Silent failure modes**: DINO-based geometry fails silently with plausible-but-wrong outputs where SIFT failed loudly. **Synthesis bet**: *combine DINOv3 features with a classical geometric consistency check as a rejection head — use the inconsistency as a self-supervised signal to fine-tune the head.* No paper does this; bridges the backbone-confidence gap that TTT3R opens and uses the "frozen-attention as signal" pattern RoMa v2 + TTT3R both exploit.
- **Composable frozen signals**: TTT3R shows frozen attention is a training signal, not just features. **Synthesis bet**: *extend TTT3R's closed-form confidence-guided learning rate to RoMa v2's dense matching head — per-match learning rate derived from backbone cross-attention alignment, no retraining.* Mixes [[chen2026_ttt3r]] + [[edstedt2025_roma-v2]].
- **Multi-task frontend collapse**: RADIOv2.5 distills 4 teachers; geometry downstream consumes features + depth + semantics. **Open**: will geometry-specific downstream heads start distilling DINOv3 + a geometry-specific teacher (e.g. Metric3Dv2 depth head features) into one backbone, repeating the RADIO pattern inside the geometry lane?

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

