---
title: Foundation Features for Geometry
type: thread
tags: [foundation-model, dinov2, dinov3, feature-matching, sfm, frozen-backbone]
created: 2026-04-15
updated: 2026-04-15
sources: [wiki/papers/oquab2023_dinov2.md, wiki/papers/simeoni2025_dinov3.md, wiki/papers/edstedt2025_roma-v2.md, wiki/papers/jang2025_pow3r.md, wiki/papers/zhang2025_feed-forward-3d-survey.md, wiki/papers/heinrich2025_radiov25.md]
status: draft
---

## Working hypothesis

Between ~2023 and 2026, **frozen self-supervised ViT features** ([[dinov2|DINOv2]], DINOv3) have displaced hand-crafted + task-trained descriptors as the feature backbone for geometric computer vision — SfM, matching, monocular depth, feed-forward reconstruction. The "foundation backbone + task head" pattern is now standard. SIFT-era assumptions (rotation/scale-invariant local descriptors, carefully engineered frontend) are losing ground to "just freeze DINOv2 and regress."

## Evidence

### Feed-forward 3D reconstruction
- [[dust3r|DUSt3R]], [[mast3r|MASt3R]], [[vggt|VGGT]], [[jang2025_pow3r|Pow3R]], CUT3R — all use DINOv2 as a frozen backbone and attach a pointmap / pose / matching head on top. See [[feed-forward-structure-from-motion]] for the sister thread focused on the pipeline side.

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

