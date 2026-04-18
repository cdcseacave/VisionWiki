---
title: VLM Reasoning over 3D Scenes
type: thread
tags: [3dgs, vlm, inference-time-reasoning, compositional-query, gaussexplorer]
created: 2026-04-18
updated: 2026-04-18
sources: [wiki/papers/kim2026_gauss-explorer.md]
operating_points: [op:default]
status: draft
---

## Goal

Given a 3D scene (Gaussians, voxels, or mesh) already reconstructed by an upstream pipeline + a natural-language query, answer the query by retrieving / synthesizing evidence views and handing them to a pretrained VLM — with no per-scene feature distillation and no specialist reasoning model. The thread's ambition is to make the 3D representation a *queryable scene database* for off-the-shelf VLMs.

## Goal contract (optional, structured)

```yaml
metric: [compositional-qa-accuracy, query-latency-s, preference-score-vs-feature-lifted-baseline]
target_regime: [per-scene-reconstruction-already-done, inference-time-only-no-training, compositional-natural-language-queries]
constraints: [pretrained-vlm-accessible, no-per-scene-finetune, rendered-view-quality-sufficient]
required_capabilities: [view-retrieval, novel-view-synthesis, vlm-compositional-reasoning]
```

## SOTA pipelines

### op:default

Inference-time pipeline given a pre-trained 3DGS scene + a text query:

- **n1** · stage(s): `[[vlm-reasoning.view-retrieval]]` · filler: top-K retrieval + information-gain scoring over pre-captured + synthesized views. Source: [GaussExplorer (Kim 2026)](wiki/papers/kim2026_gauss-explorer.md). Idea: [[vlm-view-retrieval-reasoning-3dgs_kim2026]].
- **n2** · stage(s): `[[vlm-reasoning.novel-view-adjustment]]` · filler: iterative NVS in the neighborhood of retrieved views, scored by information gain until the query is answerable. Source: [GaussExplorer (Kim 2026)](wiki/papers/kim2026_gauss-explorer.md). Idea: [[vlm-view-retrieval-reasoning-3dgs_kim2026]]. Upstream: [n1]. Downstream: [n3].
- **n3** · stage(s): `[[vlm-reasoning.compositional-answering]]` · filler: off-the-shelf VLM (e.g. GPT-4V-class or open Llama-VLM) consumes the evidence view set and returns a textual answer. Idea: [[vlm-view-retrieval-reasoning-3dgs_kim2026]] (mechanism is shared across the chain — the idea spans all three stages as a multi-stage-collapse). Upstream: [n2].

## Pipeline lineage

- 2026 · op:default · initial pipeline: 3-stage retrieve → adjust → VLM reason. Driver: [[kim2026_gauss-explorer]].

## Candidate components / not yet integrated

- **SAM 3 exemplar prompts** ([[carion2026_sam-3]]) — 2D exemplars could replace text prompts for visual queries ("find things that look like this"). Bridge to `op:default` requires a composition step not yet in any paper.
- **RADIOv2.5-distilled VLM evidence** — replace the full VLM with a RADIOv2.5 student finetuned for QA. Speculative; no paper does this.

## Open questions & synthesis bets

### Bet #021 — Feature-lifted + view-retrieval hybrid
status: proposed
combines: [[vlm-view-retrieval-reasoning-3dgs_kim2026]], [[langsplat-per-scene-autoencoder_qin2024]]
stage_target: vlm-reasoning.view-retrieval
op_target: op:default
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: Use LangSplat's per-Gaussian language latents as a cheap first-pass filter ("where in the scene could the query possibly hit?") before GaussExplorer's retrieval+NVS kicks in. Cheap + specific pre-filter → fewer VLM calls → faster queries.
expected_gain: 2-5× query-latency reduction on simple queries where feature-lifted answer suffices; maintained accuracy on compositional queries via fallback to GaussExplorer's full loop.
risk: The hybrid adds architectural complexity; tuning the hand-off threshold is non-trivial.
validating_experiment: Two-stage system: LangSplat per-Gaussian score → if confidence high, return; else fall back to GaussExplorer. Ablate on compositional QA benchmark.
triggers: [ingest-of-idea:query-routing-feature-vs-vlm]
created: 2026-04-18 · updated: 2026-04-18

## Capability gaps

- **Real-time VLM evidence aggregation** — VLM inference dominates runtime; any speedup (small-VLM distillation, cached reasoning) would make the pipeline deployable. Search target: VLM distillation papers + VLM caching for 3D-scene queries.
- **Compositing reconstructed objects into existing scenes** — generative 3D can synthesize objects but inserting them into GaussExplorer's scene for "what-if" queries is untreated. Search target: 3D scene editing with relighting and shadow consistency.
- **Embodied planning** — compositional QA tested; long-horizon planning ("how do I get from A to B given this scene?") unevaluated. Search target: embodied-agent benchmarks built on 3DGS.

## Contradictions & tensions

- **Feature-lifted vs. inference-time reasoning**: [[lifting-foundation-models-to-3d]] threads' `op:per-scene-3dgs` argues per-scene feature distillation is the right lane; this thread argues the opposite. Resolution hypothesis: feature-lifted wins on simple lookup queries (fast, cheap); GaussExplorer wins on compositional reasoning. The harmonizing bet is Bet #021.

## Shelved bets / known non-compositions

(none yet)

## Sources

- [kim2026_gauss-explorer.md](../papers/kim2026_gauss-explorer.md)
