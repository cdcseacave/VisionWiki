---
title: "MVSNet: Depth Inference for Unstructured Multi-view Stereo"
type: paper
tags: [mvs, depth, cost-volume, deep-learning]
created: 2026-04-14
updated: 2026-04-14
sources: []
local_paper: papers/mvs-depth/yao_2018_mvsnet.pdf
url: https://arxiv.org/abs/1804.02505
status: stable
---

📄 [Full paper](../../papers/mvs-depth/yao_2018_mvsnet.pdf) · [arXiv](https://arxiv.org/abs/1804.02505)

## TL;DR

Yao, Luo, Li, Fang & Quan (ECCV 2018) introduce **MVSNet**, the first end-to-end deep network for [[multi-view-stereo]]. It builds a differentiable cost volume via plane-sweep homography warping into the reference frustum and regresses a depth map — a template that defines the "learned MVS" paradigm for the following five years.

## Problem

Classical PatchMatch MVS (e.g. [[colmap|COLMAP]]'s MVS stage) was the accuracy leader but slow, hand-tuned, and inflexible. Prior deep MVS attempts worked only on rectified stereo pairs; no method generalized to arbitrary N-view inputs with unknown poses per scene.

## Method

- **Feature extraction**: shared 2D CNN extracts feature maps from all input views.
- **Plane-sweep cost volume**: discretize depth into D hypotheses; for each, warp source features into the reference camera via differentiable homography, accumulate into a 4D cost volume (H×W×D×C).
- **Variance-based cost**: reduce N warped feature volumes to one by per-voxel variance across views — arbitrary-N, permutation-invariant.
- **3D CNN regularization** → soft-argmin depth regression over D hypotheses.
- **Depth map refinement** using the reference color image.

## Results

- On DTU: **significantly beats** classical MVS (Gipuma, COLMAP) on accuracy + completeness, **several × faster**.
- Ranks first on Tanks-and-Temples (April 2018) without fine-tuning — generalization from indoor (DTU) to outdoor (T&T).

## Why it matters

MVSNet defined the learned MVS template: **features → plane-sweep cost volume → 3D-CNN regularization → depth regression**. Every subsequent learned MVS (R-MVSNet, CasMVSNet, UCSNet, PatchmatchNet, etc.) is a variation on this skeleton. Also an important conceptual precursor to depth-map based radiance-field methods and to feed-forward pointmap methods like [[dust3r|DUSt3R]] that abandon explicit cost volumes.

## Relation to prior work

- Learned replacement for [[colmap|COLMAP]]'s PatchMatch MVS.
- Differentiable plane-sweep was not new (DeepStereo, DeepMVS) but MVSNet's variance aggregation and 3D-CNN regularization made it practical for arbitrary N.
- Eventually subsumed at the high end by transformer-based and foundation-feature MVS.

## Open questions / limitations

- Memory cost of the 4D volume scales poorly with resolution and depth range (addressed by R-MVSNet, CasMVSNet).
- Assumes known, accurate poses (typically from [[colmap|COLMAP]] SfM) — doesn't do joint pose estimation.

## References added to the wiki

- [[multi-view-stereo]] (concept stub updated).
