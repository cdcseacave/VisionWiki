---
title: Delaunay triangulation with Gaussians as differentiable pivots (MILo)
type: idea
source_paper: wiki/papers/guedon2025_milo.md
also_in: []

scope: topology-rewrite
stages: [mesh-reconstruction.extraction]
collapses: []
splits_into: []
rewrites: {replaces: [radiance-fields.training, mesh-reconstruction.extraction], introduces: [radiance-fields.training-with-mesh-in-loop, mesh-reconstruction.delaunay-in-loop]}

inputs: [3d-gaussians-training-state]
outputs: [differentiable-mesh-at-every-step, final-mesh]
assumptions: [static-scene, gaussian-representation-compatible, commercial-license-blocked]
requires_upstream_property: [gaussians-trained-with-appropriate-base-pipeline]
requires_downstream_property: [consumer-accepts-mesh-output]
learned_params: [gaussian-parameters-including-position]
failure_modes: [dense-variant-requires-more-memory, 3x-longer-training-than-CoMe]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, mesh-in-loop, delaunay, differentiable-mesh]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

At every training step, build a Delaunay triangulation over the Gaussian centers (acting as pivots). The triangulation is re-constructed (not re-computed incrementally — simpler). A bidirectional consistency framework: mesh-level losses (rendered from the triangulation) backprop to Gaussian parameters, so Gaussians and mesh encode the *same* surface. An SDF-from-Gaussians computation allows precise extraction without surface erosion.

## Why it wins

10× fewer mesh vertices at comparable/better geometry vs. TSDF+MC post-hoc extraction. Optimizes the final mesh directly rather than a volumetric proxy — no silent drift between representation and output.

## Preconditions & compatibility

Dense variant is memory-hungry. Training is ~3× longer than CoMe's post-hoc approach — this is the trade-off CoMe exploits (Paradigm A is faster but needs a post-extract pass). Bet #004 (CoMe confidence as Delaunay vertex weight) is the natural cross-paradigm composition. Commercial license blocked.

## Pipeline-shape implications

Topology-rewrite scope: the pipeline's shape changes from `training → extraction` (two separate stages) to `training-with-mesh-in-loop` (one fused stage). A thread adopting MILo cannot also have a post-hoc extraction node — the stages collapse.

## Trade-offs vs. the decomposed pipeline

A decomposed pipeline lets you swap the mesh-extraction algorithm (TSDF, Poisson, Delaunay) independently of training. MILo fuses them: changing the extractor means changing the loss function. Gain: no Gaussian-mesh drift. Loss: per-scene ability to tune extractor separately.
