---
title: Generative 3D from 2D Priors
type: thread
tags: [generative-3d, single-image, flow-matching, sam-3d, latent-diffusion]
created: 2026-04-18
updated: 2026-04-18
sources: [wiki/papers/chen2025_sam-3d.md]
operating_points: [op:default]
status: draft
---

## Goal

Produce 3D scene geometry + texture + per-object layout from severely under-constrained input (single RGB image, or image + minimal prompts) by combining learned 2D/3D priors with generative modeling — rather than reconstructing from multi-view photometric consistency. Success metric is human-preference fidelity and downstream usability (asset creation, sparse-view reconstruction priors), not photometric accuracy against a specific capture.

## Goal contract (optional, structured)

```yaml
metric: [human-preference-rate-vs-baseline, per-object-geometry-quality, scene-layout-plausibility]
target_regime: [single-image-input, cluttered-natural-image, static-scene]
constraints: [per-scene-optimization-acceptable, commercial-license-tracked-per-paper]
required_capabilities: [per-object-shape-generation, per-object-texture-generation, scene-layout-prediction, real-image-domain-coverage]
```

## SOTA pipelines

### op:default

Single-image → 3D scene:

- **n1** · stage(s): `[[generative-3d.shape-generation]]` · filler: [[two-stage-latent-flow-matching-scene_chen2025]]. Source: [SAM 3D (Chen 2025)](wiki/papers/chen2025_sam-3d.md). Per-object latent → decoded geometry + texture via flow-matching. Upstream: single RGB image. Downstream: [n2].
- **n2** · stage(s): `[[generative-3d.scene-layout]]` · filler: [[two-stage-latent-flow-matching-scene_chen2025]] (layout module — shared idea across stages, hence composite). Predicts per-object placement (pose + scale). Upstream: [n1].
- **n3** · stage(s): `[[generative-3d.training-data-curation]]` · (training-time, not inference) · filler: [[real-image-data-engine_chen2025]]. Human/model-in-the-loop data engine producing real-image 3D annotations that unlock generalization. Bundle constraint: `co_requires` the data engine.

## Pipeline lineage

- 2025 · op:default · initial pipeline: two-stage latent flow-matching + data engine. Driver: [[chen2025_sam-3d]].

## Candidate components / not yet integrated

- **SAM 3 masks as per-object conditioning** ([[carion2026_sam-3]]) — the naming of SAM 3D suggests future composition; the paper doesn't yet demonstrate it. Replaces/augments: arbitrary per-object segmentation from the input image.
- **DINOv3 as the image-feature frontend** ([[simeoni2025_dinov3]]) replacing whatever frozen backbone SAM 3D uses — unified feature stack across the generative + reconstructive threads.
- **DUSt3R/VGGT as a reconstructive prior** for the shape generator — when a second view is available, condition on MASt3R features rather than generating freely.

## Open questions & synthesis bets

### Bet #022 — SAM 3 masks as per-object conditioning into SAM 3D
status: proposed
combines: [[two-stage-latent-flow-matching-scene_chen2025]], [[sam3-native-video-ids_carion2026]]
stage_target: generative-3d.shape-generation
op_target: op:default
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: SAM 3 already produces per-concept masks + instance IDs; routing these into SAM 3D's per-object shape generator as explicit conditioning should make the "which things to generate" decision explicit rather than emergent, improving cluttered-scene performance.
expected_gain: Cleaner object-boundary generation; measurable human-preference lift on scenes with 5+ objects where the baseline SAM 3D over/under-segments.
risk: SAM 3 masks are 2D; lifting to 3D inherits the single-image-ambiguity problem. The two models' data distributions may not align.
validating_experiment: Inject SAM 3 masks as Stage-1 conditioning; compare vs. baseline SAM 3D on a cluttered-scene QA benchmark.
triggers: [ingest-of-idea:per-object-conditioned-3d-generation]
created: 2026-04-18 · updated: 2026-04-18

### Bet #023 — Generative 3D as a sparse-view prior for reconstruction
status: proposed
combines: [[two-stage-latent-flow-matching-scene_chen2025]], [[per-gaussian-self-supervised-confidence_radl2026]]
stage_target: radiance-fields.initialization
op_target: op:default (and cross-thread to radiance-field-evolution op:quality-per-scene)
confidence: low
magnitude: paradigm
cost: months
breakage_risk: high
hypothesis: Use SAM 3D's single-image generative output as an *initialization* for a sparse-view 3DGS reconstruction (1-2 views). CoMe-style confidence gates where the generative prior contributes vs. the photometric loss takes over. Could make sparse-view 3DGS work in regimes where current methods (pixelSplat, MVSplat) fail.
expected_gain: Sparse-view 3DGS benchmarks (1-3 views) — first method to combine a generative 3D prior with confidence-weighted photometric fitting.
risk: Generative priors at single-image fidelity may poison the photometric loss on conflicts; the interaction is untested.
validating_experiment: Hybrid pipeline on DL3DV sparse-view subset.
triggers: [ingest-of-idea:generative-reconstruction-bridge]
created: 2026-04-18 · updated: 2026-04-18

## Capability gaps

- **Reconstruction-vs-generation disentanglement** — no principled way to know when the output is reconstructed from the input image vs. hallucinated. Would unlock: credibility for downstream asset creation. Search target: uncertainty estimation in flow-matching / diffusion 3D generators.
- **Compositing with existing scene** — generated objects need lighting + shadow consistency with their host scene. Would unlock: scene editing pipelines. Search target: relighting-aware 3D generation.
- **Bridge to multi-view reconstruction** — when 2+ views are available, the generative prior should yield gracefully to photometric fitting. Would unlock: unified generative-reconstructive pipeline. Search target: Bet #023's hypothesis becoming a paper.

## Contradictions & tensions

- **Reconstruction vs. generation paradigm**: this thread's generative approach is orthogonal to every paper in [[radiance-field-evolution]] and [[feed-forward-structure-from-motion]], which rely on multi-view photometric consistency. No paper explicitly contrasts the two regimes on the same benchmark.

## Shelved bets / known non-compositions

(none yet)

## Sources

- [chen2025_sam-3d.md](../papers/chen2025_sam-3d.md)
