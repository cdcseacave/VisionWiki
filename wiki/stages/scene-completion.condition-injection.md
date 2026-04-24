---
title: Scene-completion condition-injection
type: stage
slug: scene-completion.condition-injection
consumes: [pretrained-generative-prior, query-partial-scan-condition, optional-layout-condition]
produces: [completed-tsdf-volume]
invariants: [observed-region-fidelity-preserved, generative-prior-not-degraded-by-fine-tuning]
provides_properties: [completion-respects-input-observed-geometry, multi-sample-completion-diversity]
requires_upstream_properties: [generative-prior-supports-frozen-backbone-with-side-channel-conditioning, partial-scan-encoder-shared-with-generator]
data_regime: [partial-scan-input, optional-layout-input, indoor-or-outdoor]
tags: [scene-completion, controlnet, conditioning, frozen-backbone, self-supervised-fine-tuning]
created: 2026-04-24
updated: 2026-04-24
---

## What it is

Takes a pretrained generative scene prior and a query partial scan, and produces a completed scene that (a) preserves the observed regions of the query, (b) plausibly synthesizes the unobserved regions using the generative prior, and (c) optionally respects auxiliary conditioning (layout boxes, text).

This is the stage where the generative prior becomes a *completion* model. The key design tension: how to inject the partial-scan conditioning without (i) overwriting the learned scene prior via catastrophic fine-tuning forgetting, and (ii) requiring complete ground-truth meshes for completion supervision (which would defeat the entire partial-data-training premise).

## Example fillers

- [[controlnet-frozen-flow-self-supervised-completion_meng2026]] — ControlNet branch initialized from the frozen pretrained generator + frame-drop self-supervised pretext task. Trains the conditioning branch only; generator weights stay frozen. Requires [[visibility-guided-masked-flow-matching_meng2026]] + [[visibility-aware-masked-sparse-vae_meng2026]] upstream.

## Notes on valid fillers

Valid fillers must:
1. Avoid catastrophic forgetting of the pretrained generative prior — typically by freezing the backbone + adding parallel branches, but in-context conditioning or LoRA-style adaptation could also work.
2. Provide a self-supervised pretext task or have access to (partial, more-partial) training pairs without requiring complete GT.
3. Preserve the upstream visibility-aware encoding semantics (any condition encoder used for partial scans should distinguish observed from unobserved).

Full fine-tuning of the generator (with partial-vs-more-partial pairs) is *not* a valid filler — it risks degrading the broader generative prior on regions where partial-scan-conditioning gives no signal.

For house-scale (multi-patch) completion, this stage typically also includes a patch-fusion sub-step (weighted overlap averaging in Seen2Scene). A separate stage page may be warranted if alternative fusion strategies emerge.
