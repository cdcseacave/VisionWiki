---
title: LLM-Native Structured Scenes
type: thread
tags: [llm, point-cloud, structured-output, scene-layout, spatiallm]
created: 2026-04-18
updated: 2026-04-18
sources: [wiki/papers/mao2025_spatiallm.md]
operating_points: [op:default]
status: draft
---

## Goal

Produce structured, machine-readable scene descriptions (walls, doors, windows, oriented bounding boxes — Python-like scripts) from point-cloud or mesh input by leveraging a fine-tuned LLM as the scene-understanding frontend, rather than specialist architectures (RoomFormer, V-DETR, SceneScript). The thread's ambition is to unify 3D layout, detection, and categorization behind one LLM output channel.

## Goal contract (optional, structured)

```yaml
metric: [structured3d-layout-iou2d, per-category-AP, open-vocab-coverage]
target_regime: [point-cloud-input-rgbd-or-slam, indoor-scenes-default, closed-or-open-vocab]
constraints: [llm-fine-tune-budget-available, synthetic-pretraining-OK]
required_capabilities: [point-cloud-tokenization, llm-structured-decoding, synthetic-to-real-transfer]
```

## SOTA pipelines

### op:default

Point cloud → structured scene script:

- **n1** · stage(s): `[[llm-structured-scenes.point-cloud-tokenization]]` · filler: Sonata encoder producing 2.5cm voxel tokens. Source: [SpatialLM (Mao 2025)](wiki/papers/mao2025_spatiallm.md). Idea: [[sonata-llm-structured-scene-scripts_mao2025]]. Upstream: point cloud from RGBD scan, MASt3R-SLAM, or LiDAR. Downstream: [n2].
- **n2** · stage(s): `[[llm-structured-scenes.llm-decoding]]` · filler: fine-tuned Qwen2.5-0.5B emitting Python-like scene scripts (walls, doors, windows, oriented bboxes). Idea: [[sonata-llm-structured-scene-scripts_mao2025]]. Upstream: [n1].
- **n3** · stage(s): `[[llm-structured-scenes.pretraining-data]]` · (training-time, not inference) · filler: [[large-scale-synthetic-indoor-dataset_mao2025]] — 12,328 scenes / 54,778 rooms. Bundle constraint: `co_requires`.

## Pipeline lineage

- 2025 · op:default · initial pipeline: Sonata + Qwen2.5 fine-tune + structured scene scripts. Driver: [[mao2025_spatiallm]].

## Candidate components / not yet integrated

- **MASt3R-SLAM reconstructions as point-cloud source** ([[murai2025_mast3r-slam]]) — zero-shot compatibility demonstrated but not integrated into a canonical capture-to-structure pipeline.
- **Open-vocabulary 3D VQA extension** — the paper flags this as future work; no paper delivers it yet.
- **SpatialLM + Gaussian-Grouping identity lifting** — combine structured-script layout with per-Gaussian instance IDs to enable "edit the chair in the dining room" queries. Not in any paper.

## Open questions & synthesis bets

### Bet #024 — MASt3R-SLAM + SpatialLM as a single monocular-to-structure pipeline
status: proposed
combines: [[sonata-llm-structured-scene-scripts_mao2025]]
stage_target: llm-structured-scenes.point-cloud-tokenization
op_target: op:default
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: Paper acknowledges zero-shot compatibility with MASt3R-SLAM reconstructions. Building this as a canonical monocular-video → structured-scene pipeline (no RGBD needed) would be the cleanest demo of "feed-forward SfM + structured LLM understanding" as a two-block pipeline with no intermediate handcrafting.
expected_gain: First monocular-video-to-scene-script pipeline; robotics + AR demos unlocked.
risk: Cross-domain gap (SLAM point clouds vs. RGBD/synthetic) acknowledged by paper — fine-tuning on MASt3R-SLAM outputs may be needed.
validating_experiment: MASt3R-SLAM on phone video → Sonata → SpatialLM; evaluate on a held-out apartment tour dataset.
triggers: [ingest-of-idea:slam-pointcloud-domain-adaptation]
created: 2026-04-18 · updated: 2026-04-18

### Bet #025 — SpatialLM structured output as Gaussian Grouping supervision
status: proposed
combines: [[sonata-llm-structured-scene-scripts_mao2025]], [[gaussian-grouping-identity-encoding_ye2024]]
stage_target: lifting-foundation-models.per-primitive-identity
op_target: op:default (cross-thread to lifting-foundation-models-to-3d op:per-scene-3dgs)
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: med
hypothesis: SpatialLM's oriented-bbox output gives per-instance 3D containers; assign all Gaussians inside a bbox the same identity ID. Eliminates Gaussian Grouping's cross-view-SAM-association entirely, replacing it with a geometric assignment from LLM-emitted bboxes.
expected_gain: Cleaner per-instance identity fields in cluttered indoor scenes; eliminates a fragile stage of Gaussian Grouping.
risk: Bbox granularity loses sub-instance structure (e.g. chair legs vs. seat vs. back). SpatialLM's closed-vocab limits which instances exist.
validating_experiment: Replace Gaussian Grouping's supervision with SpatialLM bboxes on ScanNet++; compare mIoU.
triggers: [ingest-of-idea:geometric-bbox-to-identity-assignment]
created: 2026-04-18 · updated: 2026-04-18

## Capability gaps

- **Open-vocabulary structured decoding** — current SpatialLM is closed-vocab. Would unlock: user-defined object categories without retraining. Search target: LLM-emitted scene scripts with free-form category names, validated against a scene-specific ontology.
- **Impact on base LLM capabilities** — fine-tuning a 0.5B LLM on spatial tasks may degrade its natural language ability. Not evaluated. Search target: instruction-tuned variants that keep general LLM skills.
- **Small-object recall** — picture / sink categories flagged as difficult. Search target: resolution-aware tokenization or attention over fine-grained voxels.

## Contradictions & tensions

- **Specialist models vs. one-LLM-for-all**: the thread argues the LLM lump-sum approach is the right lane. Specialist models (RoomFormer, V-DETR) still win per-category on some benchmarks; the unified claim is regime-dependent.

## Shelved bets / known non-compositions

(none yet)

## Sources

- [mao2025_spatiallm.md](../papers/mao2025_spatiallm.md)
