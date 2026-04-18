---
title: 3D spatial-consistency regularizer for per-Gaussian identity
type: idea
source_paper: wiki/papers/ye2024_gaussian-grouping.md
also_in: []

scope: drop-in
stages: [lifting-foundation-models.3d-identity-regularization]
collapses: []
splits_into: []
rewrites: {}

inputs: [3d-gaussians-with-identity, per-gaussian-depth-and-normal]
outputs: [regularization-loss]
assumptions: [local-neighborhoods-share-instance, depth-normal-boundaries-mark-instance-boundaries]
requires_upstream_property: [per-gaussian-identity-available, depth-normal-rendered]
requires_downstream_property: [tolerates-smoothing-loss]
learned_params: []
failure_modes: [over-smooths-thin-multi-instance-structure]

requires: [gaussian-grouping-identity-encoding_ye2024]
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [3dgs, identity-regularization, spatial-consistency]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

For each Gaussian, find its k nearest neighbors in 3D. Penalize disagreement between its identity and its neighbors' identities, unless the depth or normal between them crosses a learned threshold (indicating an instance boundary). The regularization drives neighboring Gaussians toward the same track ID while preserving sharp boundaries at geometric discontinuities.

## Why it wins

Without this term, per-Gaussian identities become noisy and "floaty" — single Gaussians in a table can acquire a chair's ID because they happen to project into a chair-labeled pixel in one view. The regularizer removes these artifacts by enforcing that identities respect 3D structure, not just 2D supervision. Ablation in the paper shows materially cleaner edit results (object removal leaves no speckle).

## Preconditions & compatibility

Requires access to per-Gaussian depth and normal (available from any 3DGS pipeline via the feature-channel trick). Compatible with any identity-encoding scheme. Risks over-smoothing thin structures where multiple instances legitimately coexist in a small neighborhood (e.g. chair legs adjacent to floor).

## Open questions

- Threshold tuning is per-scene; would benefit from a learned or adaptive variant.
- Interaction with SAM 3's native IDs is untested — if SAM 3 IDs are already clean, regularization may be unnecessary.
