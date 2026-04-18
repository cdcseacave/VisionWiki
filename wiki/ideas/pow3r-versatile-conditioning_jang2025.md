---
title: Pow3R — versatile conditioning (intrinsics / poses / depth) into DUSt3R
type: idea
source_paper: wiki/papers/jang2025_pow3r.md
also_in: []

scope: stage-swap
stages: [feed-forward-sfm.prior-injection]
inputs: [image-pair, optional-intrinsics, optional-poses, optional-sparse-depth]
outputs: [conditioned-pointmap-or-pose-output, third-pointmap-for-procrustes]
assumptions: [dust3r-backbone-available, per-block-mlp-injection-feasible]
requires_upstream_property: [dust3r-style-vit-backbone]
requires_downstream_property: [consumer-uses-pointmap-or-pose-output]
learned_params: [per-block-mlp-weights, modality-dropout-training-schedule]
failure_modes: [priors-contradicting-image-evidence-not-handled]

requires: []
unlocks: [third-pointmap-procrustes_jang2025]
co_requires: [random-modality-dropout_jang2025]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, dust3r, prior-injection, pointmap]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Per-block MLPs inject any subset of auxiliary priors (intrinsics, poses, sparse depth) into DUSt3R's ViT encoder/decoder. A **third pointmap output** `X^{2,2}` lets Procrustes alignment recover both cameras in a single forward pass (no PnP loop). **Sliding-window high-res inference** via crop-position encoding: the intrinsics input conveys where the crop came from.

## Why it wins

Substantially beats DUSt3R when priors exist (MVS DTU); parity when they don't — single model for all prior combinations. Orders of magnitude faster pose recovery than DUSt3R's PnP loop. Bundles with modality dropout (`co_requires:`) so inference can accept any subset of priors.

## Preconditions & compatibility

Natural synthesis bet: inject RoMa v2's covariance-weighted sparse depth as Pow3R's depth prior — the combination should strictly improve DTU numbers but no paper tries it. Unlocks the third-pointmap Procrustes idea (`unlocks:`).

## Open questions

- What happens when priors contradict the image evidence? No graceful-degradation scheme defined.
