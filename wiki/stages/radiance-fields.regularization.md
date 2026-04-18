---
title: Radiance-fields regularization
type: stage
slug: radiance-fields.regularization
consumes: [3d-primitives, multiview-rendered-signals]
produces: [regularization-loss-terms]
invariants: [does-not-interfere-with-primary-photometric-convergence]
provides_properties: [surface-adherent-primitives, view-consistent-geometry]
requires_upstream_properties: [multi-view-rendering-available]
data_regime: [static-scene, posed-input, ≥2-views-overlap]
tags: [3dgs, regularization, variance-loss, planar-constraint]
created: 2026-04-18
updated: 2026-04-18
---

## Description

Apply constraints on the 3D primitives themselves — planar alignment, normal consistency, variance-across-views, edge-aware smoothness — to drive them onto surfaces and away from spurious configurations (floaters, off-surface Gaussians). Orthogonal to loss-balancing; runs alongside the primary photometric loss.

## Example fillers

- [[per-primitive-variance-losses_radl2026]] — color + normal variance across views.
- PGSR planar constraint — flatten Gaussians into planes.
- VA-GS alignment losses — edge-aware + visibility-aware + normal + feature alignment.

## Valid-filler notes

Fillers must contribute an additive loss term; fillers that alter the primitive representation itself (e.g. forcing Gaussians into 2D disks) are stage-swaps on `radiance-fields.primitives` and do not belong here. Over-regularization can kill fine structure; fillers should document failure modes.
