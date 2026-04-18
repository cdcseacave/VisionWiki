---
title: Per-pixel probabilistic view selection for PatchMatch MVS
type: idea
source_paper: wiki/papers/schonberger2016_colmap-mvs.md
also_in: []

scope: stage-swap
stages: [mvs.view-selection]
inputs: [posed-images, per-pixel-photometric-cost, per-pixel-geometric-cost]
outputs: [per-pixel-source-view-subset, depth-map-per-reference]
assumptions: [static-scene, posed-input, lambertian-default]
requires_upstream_property: [poses-metric-or-up-to-scale]
requires_downstream_property: [tsdf-or-mesh-fusion-accepts-per-pixel-depth]
learned_params: []
failure_modes: [textureless-non-lambertian-surfaces, extreme-exposure-variation]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [mvs, patchmatch, view-selection, classical]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

Over the PatchMatch propagation loop, maintain a probabilistic graphical model over per-pixel view-inclusion binary variables. The joint cost fuses photometric consistency, geometric consistency (cross-view depth agreement), and the view-inclusion prior. Each pixel dynamically picks its best-supporting subset of source views as optimization progresses — rather than a fixed N-neighbor heuristic.

## Why it wins

On internet photo collections with heterogeneous view overlap, source-view selection is the dominant robustness factor — the paper's ablation on ETH3D shows the view-selection model is what separates COLMAP-MVS from vanilla PatchMatch. A 2016 result that still outperforms learned MVS on accuracy-critical benchmarks a decade later.

## Preconditions & compatibility

Purely photometric — degrades on textureless / non-Lambertian surfaces without prior injection. The classical reference against which learned MVS (MVSNet family, [[yao2018_mvsnet]]) competes on speed, not accuracy.

## Open questions

- Do learned priors (mono depth, DINO features) improve the view-selection term where classical photometry fails?
- No dynamic-scene handling.
