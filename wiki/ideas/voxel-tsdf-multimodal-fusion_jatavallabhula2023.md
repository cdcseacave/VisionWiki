---
title: Voxel-TSDF online fusion with CLIP + DINO + AudioCLIP features (ConceptFusion)
type: idea
source_paper: wiki/papers/jatavallabhula2023_conceptfusion.md
also_in: []

scope: stage-swap
stages: [lifting-foundation-models.online-voxel-feature-fusion]
inputs: [rgbd-stream, sam-masks-per-frame, clip-dino-audioclip-encoders]
outputs: [per-voxel-tsdf-plus-per-voxel-feature-mean]
assumptions: [rgbd-input, online-streaming, voxel-hashing-budget]
requires_upstream_property: [sam-masks-per-frame, multi-modal-encoders-available]
requires_downstream_property: [consumer-does-cosine-query-on-voxel-features]
learned_params: []
failure_modes: [voxel-resolution-memory-tradeoff, no-novel-view-synthesis, view-invariant-vs-dependent-not-disentangled]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [tsdf, online-fusion, multimodal, clip, audioclip, robotics]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Per frame: SAM masks → crop each masked region → encode with CLIP / DINO / AudioCLIP → assign per-pixel feature vectors. Voxel-hashed TSDF fusion extended: classical Curless-Levoy running average applied to the FM features per voxel alongside SDF. Query time: text / image / audio / click → shared embedding space → cosine similarity → retrieve matching voxels.

## Why it wins

One-pass online mapping (robotics-grade latency). Multimodal queries from a single fusion pipeline — the only lifting pipeline handling audio natively. Opens the robotics branch of the lifting space that 3DGS methods don't touch.

## Preconditions & compatibility

No novel-view synthesis — purely semantic map. Voxel resolution trades memory vs. detail. Bet #016 composes ConceptFusion + RADIOv2.5 (unified teacher) + VPGS-SLAM progressive submaps for online city-scale multimodal 3D.

## Open questions

- Feature fusion is a per-voxel running average — no disentanglement of view-dependent (specular highlight features) vs. view-invariant (semantic class) signals.
- Loop-closure re-voxelization invalidates accumulated features.
