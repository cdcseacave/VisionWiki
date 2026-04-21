---
title: Feed-Forward 3D Problem Axes
type: concept
tags: [feed-forward, taxonomy, problem-axes, cross-cutting-vocabulary]
created: 2026-04-21
updated: 2026-04-21
sources: [wiki/papers/wang2026_feed-forward-3d-scene-modeling.md, wiki/papers/zhang2025_feed-forward-3d-survey.md]
status: draft
---

## What it is

A cross-cutting vocabulary for classifying feed-forward 3D reconstruction methods (and the [[wiki/ideas|ideas]] extracted from them) by the **engineering problem** each one addresses, independent of the 3D output representation (NeRF / 3DGS / Pointmap / mesh / SDF). Five axes, drawn from [[wang2026_feed-forward-3d-scene-modeling]] §4 and cross-checked against existing wiki coverage:

1. **Feature Enhancement** — quality of the implicit representation that gets decoded into 3D. Sub-axes: architecture choice, cross-view fusion, integration of visual foundation models.
2. **Geometry Awareness** — robustness and accuracy of the geometric reasoning. Sub-axes: explicit geometric aggregation (cost volumes, epipolar attention), post refinement of primitives, pose-free reconstruction.
3. **Model Efficiency** — compute / memory footprint under fixed accuracy. Sub-axes: feature efficiency (token merging, sparse attention, KV budget), representation compaction (Gaussian pruning, opacity control, deduplication).
4. **Augmentation Strategies** — closing the train-distribution gap. Sub-axes: data augmentation (synthetic scenes, procedural generation, SfM-bootstrapped GT), visual augmentation (generative priors fixing artifacts / hallucinating occluded regions).
5. **Temporal-aware Models** — extension to 4D. Sub-axes: online streaming (causal, per-frame updates), offline processing (non-causal, full-window aggregation), interactive (user-in-the-loop physics / editing).

These axes are **orthogonal to the output-representation axis** (NeRF / 3DGS / Pointmap / …) covered by [[zhang2025_feed-forward-3d-survey]].

## How it works

The problem axis is a labeling overlay, not a schema change. An idea page like [[fast-reciprocal-nn-matching_leroy2024]] — already typed by its `stages:` and `scope:` fields — additionally lives in a problem-axis cell: *Feature Efficiency* (reduces matching cost from `O(W²H²)` to `O(kWH)`). [[ttt3r-closed-form-confidence-lr_chen2026]] lives in *Feature Efficiency* (linear-time recurrence replacing attention) and *Temporal-aware: Online Streaming* (per-frame causal updates).

The value is in **cross-representation pattern surfacing**: two ideas with identical problem-axis labels but different `stages:` often compose (transfer the efficiency trick across representations), while ideas sharing a representation but different problem-axes almost never do.

The wiki does **not** currently enforce problem-axis tags in frontmatter (deliberate choice — one taxonomic source is not enough to commit to a schema extension). The axes are used informally in thread bodies until a second source beyond [[wang2026_feed-forward-3d-scene-modeling]] confirms the partition is stable, at which point idea frontmatter can grow a `problem_axis:` field (see CLAUDE.md §8 co-evolution).

## The orthogonality claim

[[wang2026_feed-forward-3d-scene-modeling]] asserts, without quantification, that methods built on different output representations may appear in any of the five problem axes (e.g., pose-free reconstruction exists in both pointmap form — MASt3R — and 3DGS form — NoPoSplat, Splatt3R, FreeSplatter, FLARE, RegGS). The claim is the survey's central thesis but is argued qualitatively.

**Testable operationalization for the wiki**: once idea pages carry problem-axis tags, a 5×5 problem × representation co-occurrence matrix should show substantial cell coverage on at least 4 of 5 axes. If one axis localizes to a single representation, the orthogonality claim is weaker for that axis — likely signaling either a genuine representation-specific problem or insufficient sampling.

This is a meta-audit, not a research bet. Filed as a latent lint candidate rather than a synthesis bet.

## Mapping axes to existing wiki threads

| Axis | Primary thread home | Representative wiki ideas already tagged implicitly |
|---|---|---|
| Feature Enhancement | [[foundation-features-for-geometry]], [[radiance-field-evolution]] | [[radiov25-agglomerative-distillation_heinrich2025]], [[dinov3-gram-anchoring_simeoni2025]], [[roma-v2-dinov3-dense-matcher_edstedt2025]] |
| Geometry Awareness | [[feed-forward-structure-from-motion]] (Tier 3), [[radiance-field-evolution]] | [[metric-scale-pointmap-loss_leroy2024]], [[plane-sweep-variance-cost-volume_yao2018]], [[flattened-gaussian-unbiased-depth_chen2024]] |
| Model Efficiency | [[gpu-native-sfm]], [[feed-forward-structure-from-motion]] (Tier 3 scaling) | [[fast-reciprocal-nn-matching_leroy2024]], [[ttt3r-closed-form-confidence-lr_chen2026]], [[large-chunk-ttt-fast-weights_jin2026]], [[morton-ordered-sparse-voxel-rasterizer_sun2025]] |
| Augmentation Strategies | [[generative-3d-from-2d-priors]], [[radiance-field-evolution]] | [[two-stage-latent-flow-matching-scene_chen2025]], [[large-scale-synthetic-indoor-dataset_mao2025]], [[real-image-data-engine_chen2025]] |
| Temporal-aware Models | [[feed-forward-structure-from-motion]] (Tier 3, online streaming), [[generative-3d-from-2d-priors]] (4D future) | [[ttt3r-closed-form-confidence-lr_chen2026]], [[large-chunk-ttt-fast-weights_jin2026]], [[loger-hybrid-ttt-plus-swa_zhang2025]] |

This mapping is indicative, not exhaustive. It surfaces the Model Efficiency axis as the wiki's densest zone (consistent with the TTT family getting heavy recent attention) and the Augmentation Strategies axis as the sparsest — a legitimate under-covered direction if a concrete augmentation paper lands (MegaSynth / Aug3D are candidates flagged in [[wang2026_feed-forward-3d-scene-modeling]]).

## Relation to the wiki's own stage system

The problem axis is **not a replacement** for the stage system ([[wiki/stages|wiki/stages/]]). Stages are typed slots with I/O contracts; problem axes are descriptive labels on the *design intent* of an idea that fills a slot. Two ideas can fill the same stage with different problem intents (e.g., `feature-matching.task-head` can be filled by a feature-enhancement idea or a model-efficiency idea) and compose only if their `co_requires:` allow.

Rough correspondence: *Feature Enhancement* and *Geometry Awareness* tend to drive filler quality on existing stages; *Model Efficiency* tends to live in `multi-stage-collapse` or `topology-rewrite` scope ideas; *Augmentation* and *Temporal-aware* are more likely to introduce new stages (new `stage-split` or `new-paradigm` ideas). Not a rule — a tendency.

## Open questions

- Does the orthogonality claim survive a concrete 5×5 audit of the wiki's own idea pages? The survey asserts but doesn't quantify.
- Is *Augmentation Strategies* really one axis, or does the Data / Visual split in §4.4 belong on two separate axes? Data augmentation (training-time) and visual augmentation (inference-time generative polish) serve quite different design intents.
- Where does the **reconstruction-vs-generation spectrum** (§7.6 of [[wang2026_feed-forward-3d-scene-modeling]]) sit in this taxonomy? It reads as a cross-cutting *dial* (how much to hallucinate) rather than an axis — possibly a sixth parameter worth tracking once more data lands.

## Key references

- [[wang2026_feed-forward-3d-scene-modeling]] — origin of the 5-axis taxonomy.
- [[zhang2025_feed-forward-3d-survey]] — orthogonal (representation-axis) taxonomy for the same field.
