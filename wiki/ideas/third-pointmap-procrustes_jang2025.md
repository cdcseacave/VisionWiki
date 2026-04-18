---
title: Third-pointmap output + Procrustes for single-pass two-camera recovery
type: idea
source_paper: wiki/papers/jang2025_pow3r.md
also_in: []

scope: drop-in
stages: [feed-forward-sfm.pose-recovery]
inputs: [dust3r-style-pointmaps, third-pointmap-x22]
outputs: [two-camera-poses-single-pass]
assumptions: [pointmaps-metric-or-consistent-scale]
requires_upstream_property: [pow3r-or-equivalent-third-pointmap-output]
requires_downstream_property: []
learned_params: []
failure_modes: [procrustes-sensitive-to-outliers-in-pointmap]

requires: [pow3r-versatile-conditioning_jang2025]
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [feed-forward-sfm, pose-recovery, procrustes]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

DUSt3R outputs pointmaps `X^{1,1}` (self) and `X^{1,2}` (other-in-self-frame). Pow3R adds a third pointmap `X^{2,2}` (other-in-other-frame). The matched pair `X^{1,2}` vs. `X^{2,2}` admits a closed-form Procrustes solution for the relative pose between cameras 1 and 2 — no PnP optimization loop needed.

## Why it wins

Orders of magnitude faster than DUSt3R's PnP loop. Requires only a single forward pass + a linear-algebra closed form.

## Open questions

- Outlier robustness of Procrustes vs. PnP?
