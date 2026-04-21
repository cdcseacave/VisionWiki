---
title: "Information routing vs 3D-rendering memory (video-world memory paradigms)"
type: concept
tags: [video-generation, long-horizon, 3d-memory, world-model, information-routing, lyra-2, gen3c, spmem, worldmem, vmem, context-as-memory]
created: 2026-04-21
updated: 2026-04-21
sources: [wiki/papers/shen2026_lyra2.md, wiki/papers/wang2026_feed-forward-3d-scene-modeling.md]
status: draft
---

## What it is

A **design-principle dichotomy** for how long-horizon video-world generators use 3D geometry to maintain scene persistence. The two positions:

1. **3D-as-rendering-memory** (also called "global 3D memory" or "cumulative 3D representation"): accumulate a single global 3D scene representation from past generations; render novel views from it; feed the rendered views back as conditioning for subsequent generation. Used by Gen3C, SPMem, Lyra 1, and by architectures that maintain a persistent 3D state.

2. **3D-as-information-routing** (the Lyra 2.0 framing): maintain *per-frame* geometry independently, with no fused global model. Use the geometry **solely** for (a) selecting which history frames are relevant to the current target viewpoint and (b) conveying geometric correspondence to the generative backbone. Appearance synthesis is left entirely to the pretrained video prior, which resolves minor inconsistencies between observations rather than propagating rendering artifacts from an error-prone global model.

The distinction is load-bearing because it predicts a different failure mode and a different cost profile:

| Axis | 3D-as-rendering-memory | 3D-as-information-routing |
|---|---|---|
| What the model sees | Rendered views of accumulated 3D scene | Retrieved raw history frames + geometric correspondence signals |
| Failure amplifier | Depth / pose errors accumulate into global model; errors become *baked into* the conditioning | Per-frame depth errors stay local; no accumulation mechanism |
| What the DiT prior does | Corrects rendering artifacts in 2D after the fact (if it can) | Synthesizes appearance from scratch, guided only by geometric correspondence |
| Quality of per-frame artifact propagation | High: generative artifacts degrade 3D → flawed conditioning → worse next frames | Low: correspondences are appearance-neutral; prior is never "told" to reproduce artifacts |
| Compute profile | One accumulating structure + one rendering pass per step | Per-frame cache + one retrieval + one warp per step |

The Lyra 2.0 paper documents this choice at §4.2:

> *"In contrast to global 3D memory methods that rely on a single accumulated scene representation, we maintain per-frame 3D geometry and use it exclusively for information routing, i.e., retrieving relevant history frames and establishing dense geometric correspondences, while relying on the video model's generative prior for appearance synthesis."*

## Why it matters for the wiki

This concept is the **organizing principle** for two first-class idea pages — `[[per-frame-3d-cache-retrieval_shen2026]]` and `[[canonical-coord-warp-injection_shen2026]]` — and is the axis along which Lyra 2.0 argues against Gen3C/SPMem/Lyra 1. Without a concept page, the contrast gets fragmented across prose in multiple paper pages and threads. With one, it becomes a query vocabulary: one can ask "which wiki ideas commit to rendering-memory vs routing?" as a graph query.

Ablation evidence from Lyra 2.0 Table 3 operationalizes the principle: replacing per-frame routing with global-cloud rendering loses **Camera Controllability 63.87 → 49.86** (−14.01 absolute) on Tanks-and-Temples long video generation — the largest single ablation swing in the paper.

## Related

- [[wang2026_feed-forward-3d-scene-modeling]] §4.5.1 **LongStream** mechanism (gauge-decoupled streaming visual geometry) is a geometry-for-routing variant in the streaming reconstruction setting — same principle, different domain. The [[feed-forward-structure-from-motion]] thread has this as an open question.
- [[feed-forward-problem-axes]]: the routing-vs-rendering distinction does not map cleanly onto the five Wang-2026 problem axes (feature enhancement / geometry awareness / efficiency / augmentation / temporal-awareness). It is a *design-principle axis* for the temporal-awareness family specifically, orthogonal to the five.
- [[generative-3d-from-2d-priors]]'s existing capability gap "video-world vs 3D-world paradigm choice" (introduced by Wang 2026 §7.4) is a broader umbrella; this concept refines the video-world paradigm's internal design space.

## Representative fillers across the wiki

**Routing family** (this is where new bets belong):
- Lyra 2.0 (per-frame cache + visibility retrieval + canonical-coord injection): [[per-frame-3d-cache-retrieval_shen2026]] + [[canonical-coord-warp-injection_shen2026]].
- Context-as-Memory [128], WorldMem [118]: FOV-based retrieval without geometry-aware dense correspondence. **Partial routing** — retrieval is present, correspondence injection is absent. Not yet in the wiki as idea pages.
- VMem [51]: geometry-aware retrieval on indexed 3D surface elements. Not yet in the wiki.

**Rendering-memory family**:
- Gen3C [81]: depth-warped rendered conditioning. Not yet in the wiki.
- SPMem [117]: global accumulated point cloud + rendered conditioning. Not yet in the wiki.
- Lyra 1 [2]: cumulative 3D representation as conditioning. **Stub paper page** [[bahmani2025_lyra]] (created in this ingest as a load-bearing precedent).

**Orthogonal** (not primarily about video-world memory):
- [[zhao2025_diffusionsfm|DiffusionSfM]], [[chen2026_ttt3r|TTT3R]], [[jin2026_zipmap|ZipMap]] — address reconstruction, not generation memory.

## Open questions

- Is there an intermediate paradigm (e.g., routing + a lightweight global summary for topology) that captures the retrieval benefits and the global-scene benefits?
- How does the routing-vs-rendering trade-off shift when the input is dense video (not single-image exploration)? Lyra 2.0 targets single-image-in; rendering-memory may dominate when many calibrated views are available and per-frame depth is less noisy.
- The concept currently has a single in-wiki paper (Lyra 2.0) fully committing to the routing position; a second routing-family paper in the wiki would test whether the principle generalizes.
