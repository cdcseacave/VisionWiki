---
title: Airspace-aware visibility criterion for cell partitioning
type: idea
source_paper: wiki/papers/lin2024_vastgaussian.md
also_in: []

scope: drop-in
stages: [radiance-fields.partitioning]
inputs: [ground-plane-camera-positions, per-cell-bbox, per-camera-image-projection]
outputs: [per-cell-camera-assignments]
assumptions: [flat-ish-ground-plane, manhattan-aligned-world-axes, static-scene]
requires_upstream_property: [camera-positions-available, scene-bbox-known]
requires_downstream_property: [per-cell-training-accepts-camera-assignment]
learned_params: []
failure_modes: [non-planar-scenes-fail, user-picks-m-x-n-by-hand]

requires: []
unlocks: []
co_requires: [progressive-4-step-partition_lin2024]
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, city-scale, partitioning, visibility]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Per-cell visibility is computed as `Ω_air_ij / Ω_i` — projected area of the cell's full **axis-aligned bounding box from ground to the highest point** divided by image area. The critical move: the cell's 3D extent includes airspace (floor to ceiling), not just the convex hull of surface points. A camera is assigned to the cell if its airspace visibility exceeds threshold `T_h = 25%`.

## Why it wins

Floaters live *off* surfaces — by definition, in the airspace. Surface-convex-hull visibility misses the very cameras that need to observe the airspace above each cell to prevent floater creation there. Ablation Tab. 3: airspace-agnostic visibility → PSNR 26.81 → 24.54, heavy floater artifacts (Fig. 8). The ~2 dB drop is the cleanest isolation of this idea.

## Preconditions & compatibility

Requires a ground-plane Manhattan alignment and a scene bbox. Bundles with progressive 4-step partition (`co_requires:`) — airspace visibility alone is insufficient without the region-division + position-based + coverage-based steps. Non-planar scenes (canyons, multi-story indoor) need a different partitioning primitive.

## Open questions

- No automatic `m × n` selection — user picks by hand.
- Does the criterion port to non-partition-based methods (e.g. submap-based SLAM)?
