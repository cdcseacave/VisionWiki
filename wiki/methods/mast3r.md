---
title: MASt3R
type: method
tags: [3d-reconstruction, feature-matching, feed-forward, transformer, pointmap, metric-scale]
created: 2026-04-12
updated: 2026-04-21
sources:
  - papers/leroy2024_mast3r.md
  - papers/edstedt2025_roma-v2.md
  - papers/elflein2026_vgg-t3.md
  - papers/jang2025_pow3r.md
  - papers/mao2025_spatiallm.md
  - papers/murai2025_mast3r-slam.md
  - papers/pataki2025_mp-sfm.md
  - papers/zhang2025_feed-forward-3d-survey.md
  - papers/zhao2025_diffusionsfm.md
status: draft
---

## What it is

MASt3R ("Matching And Stereo 3D Reconstruction") is a feed-forward transformer introduced by [Leroy, Cabon & Revaud 2024](../papers/leroy2024_mast3r.md) that augments [[dust3r|DUSt3R]] with a dense-descriptor head and metric-scale pointmap output. Given two uncalibrated images, it produces (i) per-pixel 3D points in a shared coordinate frame, (ii) per-pixel confidence, and (iii) per-pixel 24-dim local feature descriptors — all in a single forward pass. Image matching, calibration, pose estimation, and reconstruction become aspects of the same network.

## How it works

Architecture inherits DUSt3R: ViT-L encoder (shared weights) → intertwined ViT-B decoders with cross-attention between views → two 3D regression heads + two new descriptor heads (2-layer MLPs with GELU, output L2-normalized 24-dim per-pixel features). Trained from the public DUSt3R checkpoint on a 14-dataset mix (10 with metric GT).

Four independent mechanisms, each a first-class idea in the wiki:

1. **Joint pointmap + descriptor supervision** ([[dust3r-matching-head_leroy2024]]): InfoNCE over GT 3D correspondences trains the descriptor head; total loss `L_conf + L_match`. Regression head regularizes descriptor space; descriptor head lifts pose accuracy (median rotation 9.4° → 3.0°). Synergy confirmed bi-directionally (Table 1).
2. **Metric-scale regression** ([[metric-scale-pointmap-loss_leroy2024]]): disable the `1/z` normalization on metric-GT training batches. Enables single-forward-pass absolute-scale pose (94.1 VCRE AUC on Map-free from pointmap alone, no matching).
3. **Fast reciprocal-NN matching** ([[fast-reciprocal-nn-matching_leroy2024]]): iterative cycle-finder on descriptor maps, `O(k·WH)` vs `O(W²H²)`. 64× speedup and *better* AUC than naive reciprocal matching via implicit outlier filtering. Fills new stage [[feature-matching.reciprocal-matching]].
4. **Coarse-to-fine greedy window covering** ([[coarse-to-fine-window-covering-matching_leroy2024]]): greedy set-cover of full-resolution window pairs seeded by a 512px coarse pass; enables matching at arbitrary input resolution despite the ViT-512 training cap. 4.6× Chamfer reduction on DTU MVS zero-shot over DUSt3R.

## Variants & lineage

- **Predecessor**: [[dust3r|DUSt3R]] (Wang et al. CVPR 2024) — same backbone, no descriptor head, scale-invariant loss only.
- **Direct successors**: [[murai2025_mast3r-slam|MASt3R-SLAM]] (real-time SLAM using MASt3R priors), MASt3R-SfM (offline reconstruction pipeline, referenced by the paper).
- **Downstream consumers**: [[pataki2025_mp-sfm|MP-SfM]] (optional matcher), [[jang2025_pow3r|PoW3R]] (conditional DUSt3R/MASt3R with extra priors), [[jin2026_zipmap|ZipMap]], [[chen2026_ttt3r|TTT3R]], and [[edstedt2025_roma-v2|RoMa v2]] (comparison baseline).
- **Related competing direction**: RoMa v2 — dense matching without 3D grounding, uses DINOv3 features + custom CUDA refinement. Highest dense-matcher SOTA, but doesn't predict 3D structure.

## Strengths

- Joint 3D + descriptor output: matching, calibration, pose, reconstruction from one forward pass.
- Extreme-viewpoint robustness: 30 absolute AUC points over LoFTR+KBR on Map-free (the hardest viewpoint-change benchmark).
- Metric-scale output without external depth priors (previously the DUSt3R family's achilles heel).
- Reciprocal matcher is reusable independently of the dual-head (works on any dense matcher).
- Zero-shot generalization: top zero-shot MVS on DTU (0.374mm Chamfer) despite not training on DTU.

## Limitations

- Trained only on pinhole images — distortion degrades geometry.
- 512px max training resolution — downstream wrapper (coarse-to-fine) works, but the architecture's training recipe doesn't scale to native high-res.
- Scale inconsistency across independently-predicted pairs → downstream Sim(3) alignment often needed.
- Direct-regression pose brittle to calibration mismatch (Aachen Day top1 direct-reg: 1.5/4.5/60.7 vs matching: 79.6/93.5/98.7).
- `d = 24` descriptor dim is unusually small; no ablation isolates why this works.
- CC-BY-NC-SA-4.0 code + weights: not usable for commercial products as-is.

## Typical use in photogrammetry/ML pipelines

- **Extreme-viewpoint pose estimation** (e.g., map-free localization, re-localization across seasons): use MASt3R + fast-reciprocal-NN as matcher frontend, feed to classical pose solver.
- **Tier-3 feed-forward SfM bootstrap** ([[feed-forward-structure-from-motion]]): MASt3R produces pair-wise metric pointmaps that can be aligned into a global reconstruction — see MASt3R-SfM pipeline (referenced in the paper, not ingested separately).
- **Real-time SLAM** ([[murai2025_mast3r-slam|MASt3R-SLAM]]): MASt3R decoder runs once per keyframe; iterative pointmap matching replaces feature matching at 15 FPS.
- **Zero-shot MVS** on out-of-distribution datasets (DTU, ETH3D): MASt3R + coarse-to-fine triangulation gives strong priors without domain-specific training.

## Key references

- [Leroy et al. 2024](../papers/leroy2024_mast3r.md) · [pdf](../../papers/feature-matching/leroy_2024_mast3r.pdf) — primary.
- [Murai 2025](../papers/murai2025_mast3r-slam.md) · [pdf](../../papers/sfm-slam/murai_2025_mast3r-slam.pdf) — real-time SLAM.
- [Pataki 2025](../papers/pataki2025_mp-sfm.md) · [pdf](../../papers/sfm-slam/pataki_2025_mp-sfm.pdf) — hybrid SfM option.
- [Edstedt 2025](../papers/edstedt2025_roma-v2.md) · [pdf](../../papers/feature-matching/edstedt_2025_roma-v2.pdf) — competing direction.
- [Zhang 2025 (survey)](../papers/zhang2025_feed-forward-3d-survey.md) · [pdf](../../papers/fundamentals/zhang_2025_feed-forward-3d-survey.pdf) — taxonomy.
