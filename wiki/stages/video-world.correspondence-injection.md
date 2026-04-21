---
title: Video-world correspondence injection
type: stage
slug: video-world.correspondence-injection
consumes: [retrieved-frames, per-frame-depth, camera-intrinsics, camera-extrinsics, target-camera]
produces: [per-token-correspondence-embedding]
invariants: [appearance-neutral-signal, attention-side-injection]
provides_properties: [dense-geometric-alignment-signal-to-diffusion-backbone, preserves-pretrained-value-projection]
requires_upstream_properties: [retrieved-frames-with-per-frame-geometry, pretrained-dit-or-transformer-backbone]
data_regime: [static-scene, long-horizon-autoregressive]
tags: [video-generation, correspondence, attention-injection, diffusion-conditioning]
created: 2026-04-21
updated: 2026-04-21
---

A stage that converts per-frame 3D geometry + camera poses into a per-token conditioning signal for a pretrained video diffusion backbone, so that retrieved history frames align geometrically with the target viewpoint without the model having to infer correspondences from pixels alone.

**Why it needs to exist.** Retrieval (`[[video-world.spatial-memory]]`) determines *which* frames should be in context; this stage determines *how their spatial relation is conveyed to the backbone*. The two concerns are separable: the same retrieved set of frames can be injected via (a) warped RGB conditioning, (b) pose/ray embeddings only, (c) warped canonical coordinates (appearance-free). Option (c) is the Lyra 2.0 entry — it solves the specific failure mode where (a) causes the model to re-generate disocclusion holes and depth-boundary bleeding because the warped RGB becomes "a crutch that bypasses the generative prior."

**Valid fillers** produce a per-token embedding added to the backbone's attention computations. The invariant `appearance-neutral-signal` flags fillers that leak appearance content (warped RGB does; canonical coordinates don't). The invariant `attention-side-injection` means the filler modifies queries and/or keys but not the value projection — a deliberate choice to preserve the pretrained backbone's learned pixel distribution.

**Example fillers**:
- `[[canonical-coord-warp-injection_shen2026]]` — forward-warp `(u, v, frame-idx)` canonical coordinate maps via full-resolution depth; inject on Q/K only (Lyra 2.0).

**Compatibility**:
- Bundles with `[[video-world.spatial-memory]]` via co-require — a correspondence-injection filler is useless without a retrieval mechanism to feed it history frames.
- Orthogonal to `[[video-world.drift-mitigation-training]]`.
