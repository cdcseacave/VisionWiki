---
title: Video-world spatial memory
type: stage
slug: video-world.spatial-memory
consumes: [history-frames, per-frame-depth, camera-intrinsics, camera-extrinsics, target-camera]
produces: [N-retrieved-frames, visibility-scores]
invariants: [per-frame-independence-no-global-fusion, target-camera-viewpoint-relative]
provides_properties: [geometry-aware-retrieval, revisit-coverage-beyond-temporal-window]
requires_upstream_properties: [per-frame-depth-estimate, calibrated-poses, bounded-per-frame-geometric-drift]
data_regime: [static-scene, long-horizon-autoregressive, bounded-or-unbounded]
tags: [video-generation, long-horizon, retrieval, spatial-memory, 3d-cache]
created: 2026-04-21
updated: 2026-04-21
---

A stage that, at each autoregressive step of a long-horizon video generator, selects a bounded set of history frames whose 3D content is maximally relevant to the current target viewpoint — so that revisited regions (hundreds of frames outside the temporal context window) can still be attended to.

**Why it needs to exist as a typed stage.** Long-horizon video diffusion models have a finite temporal context; naively extending that context blows up attention cost. Prior work folded this concern into the model architecture (persistent KV caches, architectural latent-state propagation) or pushed it out of the model entirely (global accumulated 3D representations rendered back as conditioning). Calling this out as a first-class stage separates *which frames to recall* (a planning problem) from *how to convey their geometric relation* (a communication problem, handled by `[[video-world.correspondence-injection]]`) and from *the actual appearance synthesis* (the pretrained video prior's job).

**Valid fillers** should produce a ranked set of history frames whose collective geometric overlap with the target viewpoint is maximized under some selection budget `N_s`. They should not fuse history into a single model — that belongs to an adjacent design paradigm where `consumes:` includes `accumulated-3d-scene` and `provides_properties:` no longer includes `per-frame-independence`. Those belong to a separate stage (not yet needed in the wiki).

**Example fillers**:
- `[[per-frame-3d-cache-retrieval_shen2026]]` — visibility-score greedy coverage over per-frame point clouds (Lyra 2.0).

**Compatible downstream stages**: `[[video-world.correspondence-injection]]` is the natural consumer; retrieved frames without correspondence injection leave the geometric relation implicit and degrade long-range camera controllability (see Lyra 2.0 Table 3 "w/ Global Point Cloud" row — 14.01 pt Camera Controllability loss).
