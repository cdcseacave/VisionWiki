---
title: Spiky 3D Gaussian Cutter (SGC) — boundary-aware Gaussian refinement
type: idea
source_paper: wiki/papers/bao2025_seg-wild.md
also_in: []

scope: drop-in
stages: [lifting-foundation-models.primitive-refinement]
inputs: [3d-gaussians, projected-sam-masks, per-gaussian-boundary-crossing-score]
outputs: [refined-gaussians-cleanly-on-one-side-of-boundaries]
assumptions: [sam-masks-consistent-enough-for-boundary-signal]
requires_upstream_property: [sam-masks-available-per-view, gaussian-to-view-projection]
requires_downstream_property: [consumer-tolerates-modified-gaussian-set]
learned_params: []
failure_modes: [threshold-scene-dependent]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, boundary-refinement, sam, seg-wild]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Project each 3D Gaussian into each view's SAM mask. If the Gaussian crosses a mask boundary above a threshold, "cut" it — split or shrink to land cleanly on one side. Produces clean object boundaries despite inconsistent SAM masks across views.

## Why it wins

SAM masks across views disagree at edges; Gaussians that straddle those edges cause per-instance bleed. SGC explicitly removes the offenders.

## Open questions

- Threshold is hand-tuned; scene-dependent.
- Does SAM 3's native cross-view IDs obsolete the need? Paper predates SAM 3.
