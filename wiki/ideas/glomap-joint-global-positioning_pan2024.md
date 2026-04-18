---
title: Joint global positioning + outlier-aware rotation averaging (GLOMAP)
type: idea
source_paper: wiki/papers/pan2024_glomap.md
also_in: []

scope: stage-swap
stages: [sfm.global-positioning, sfm.rotation-averaging]
inputs: [view-graph, pairwise-relative-poses]
outputs: [global-camera-centers, sparse-3d-points, refined-rotations]
assumptions: [view-graph-available, noise-bounded]
requires_upstream_property: [pairwise-relative-poses-from-verification]
requires_downstream_property: [downstream-colmap-compatible-output]
learned_params: []
failure_modes: [still-cpu-bound-vs-gpu-successors]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sfm, glomap, global-sfm, classical]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Joint global positioning**: camera centers + 3D points solved in one robust joint optimization (replaces translation-averaging → BA two-stage). **Outlier-aware rotation averaging**: robust weighting on the rotation-only subproblem. **Drop-in COLMAP backend**: same frontend + file format.

## Why it wins

Matches COLMAP accuracy at 2–3 orders of magnitude faster on large collections. In [[gpu-native-sfm]]'s lineage, GLOMAP is the 2024 step between COLMAP and InstantSfM — raising the classical baseline that feed-forward methods must beat.

## Open questions

- What's the theoretical floor of joint-global-positioning wall-clock on current hardware? InstantSfM beats GLOMAP 12× on GPU; GLOMAP-on-GPU would likely close most of that gap.
