---
title: Hierarchical per-ray opacity-field resort + robust opacity formulation (SOF)
type: idea
source_paper: wiki/papers/radl2025_sof.md
also_in: []

scope: stage-swap
stages: [mesh-reconstruction.extraction]
inputs: [3d-gaussians, per-ray-opacity-samples]
outputs: [hierarchically-sorted-opacity-field, marching-cubes-ready-grid]
assumptions: [static-scene, 3dgs-trained]
requires_upstream_property: [3dgs-training-complete]
requires_downstream_property: [consumer-uses-marching-cubes-on-opacity-field]
learned_params: []
failure_modes: [hierarchical-sort-memory-overhead]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, mesh-extraction, opacity-field, sof]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Hierarchical per-ray resorting replaces Gaussian Opacity Fields' (GOF) global-sort heuristic: Gaussians are sorted per-ray in a hierarchical structure that maintains correct primitive ordering. A robust opacity-field formulation handles volumetric-to-surface transition for precise level-set extraction.

## Why it wins

3× faster training, up to 10× faster mesh extraction (30 min → 3–5 min). Correct primitive ordering → fewer meshing artifacts vs GOF. SOF is the **meshing-pipeline backend** CoMe builds on top of — CoMe's contributions sit on SOF's fast level-set extraction.

## Open questions

- Hierarchical sort memory overhead at 1M+ primitives?
