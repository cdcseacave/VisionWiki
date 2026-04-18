---
title: Dual-branch hash grid + ASG radiance decomposition (AniSDF)
type: idea
source_paper: wiki/papers/gao2025_anisdf.md
also_in: []

scope: new-paradigm
stages: [radiance-fields.primitive-representation, mesh-reconstruction.extraction]
inputs: [posed-images]
outputs: [sdf-field, diffuse-specular-separated-appearance, watertight-mesh-via-MC]
assumptions: [static-scene, posed-input, hash-grid-based-representation]
requires_upstream_property: []
requires_downstream_property: []
learned_params: [coarse-hash-grid, fine-hash-grid, asg-weights, sdf-mlp]
failure_modes: [dual-branch-memory-overhead, slower-than-3dgs-for-nvs]

requires: []
unlocks: []
co_requires: []
bridges: []
equivalent_to: []
refines: []
contradicts: []

tags: [sdf, implicit-surface, hash-grid, anisotropic-spherical-gaussian, relighting]
created: 2026-04-18
updated: 2026-04-18
status: unclaimed
validated_in: []
---

## Mechanism

**Dual hash grids** (coarse levels 4–10 + fine levels 10–16) trained jointly rather than coarse-to-fine — preserves thin structures lost by sequential coarse-to-fine training in Neuralangelo/NeuS. **Anisotropic Spherical Gaussian (ASG) radiance decomposition** explicitly separates diffuse + specular branches following the rendering equation — physics-based inductive bias prevents the view-dependent color MLP from baking specular into geometry.

## Why it wins

Best Shiny-Blender / luminous-object reconstruction. Thin structures retained. Direct mesh via MC on the SDF. Fills the implicit-SDF lane that 3DGS methods don't address — orthogonal to [[per-gaussian-discretized-sdf_zhu2025]]'s approach (per-Gaussian SDF samples vs. continuous MLP SDF).

## Open questions

- Can the coarse/fine joint training idea transfer to 3DGS densification schedules?
