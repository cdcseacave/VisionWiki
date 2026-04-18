---
title: Closed-form Gaussian Shannon MI for real-time 3DGS next-best-view (GauSS-MI)
type: idea
source_paper: wiki/papers/xie2025_gauss-mi.md
also_in: []

scope: new-paradigm
stages: [active-reconstruction.uncertainty-representation, active-reconstruction.next-best-view-scoring]
inputs: [3dgs-scene-in-progress, candidate-novel-viewpoint]
outputs: [information-gain-scalar, ranked-viewpoints]
assumptions: [active-capture-robot-available, real-time-budget, gaussian-color-variance-representable]
requires_upstream_property: [3dgs-expressing-per-gaussian-color-variance]
requires_downstream_property: [motion-planner-uses-ranked-viewpoints]
learned_params: [per-gaussian-sh-variance]
failure_modes: [variance-training-adds-cost, closed-form-assumes-gaussian-posterior]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, active-reconstruction, mutual-information, next-best-view]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Probabilistic per-Gaussian color variance**: SH coefficients treated as random variables with learned mean + variance. **Closed-form Shannon MI**: analytical expected information gain per novel viewpoint, propagated through differentiable rendering — no Monte Carlo sampling. Motion planner scores candidate viewpoints by MI, picks the best.

## Why it wins

Real-time NBV at closed-form cost. PSNR 34.35 on Oil Drum in 141 frames vs FUEL's 22.82 in 165. Prior MI-based methods (FisherRF) aren't real-time; prior real-time methods (FUEL) lack visual quality.

## Preconditions & compatibility

Synthesis bet: use GauSS-MI's color variance as CoMe's confidence source — both are per-Gaussian uncertainty measures, CoMe uses it for loss balancing, GauSS-MI for NBV. Unified uncertainty model would serve both.

## Open questions

- Closed-form MI assumes Gaussian posterior; non-Gaussian uncertainty (multimodal appearance) unevaluated.
