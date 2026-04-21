---
title: Forward-backward depth consistency filter
type: stage
slug: sfm.forward-backward-depth-consistency
consumes: [refined-per-view-depth-maps, depth-uncertainties, candidate-registration]
produces: [accept-or-reject-registration-decision, inconsistent-view-tags]
invariants: [symmetry-induced-misregistration-rejected]
provides_properties: [rejects-ghost-cameras-due-to-symmetry, uses-dense-geometry-not-raw-images]
requires_upstream_properties: [per-pixel-depth-uncertainty, refined-depth-maps-co-estimated-with-poses]
data_regime: [incremental-sfm, post-registration-filter]
tags: [sfm, filtering, symmetry, mp-sfm]
created: 2026-04-21
updated: 2026-04-21
---

Post-registration filter that rejects mis-registered views — typically from symmetric or repetitive structures — by reprojecting refined depth maps across overlapping views and measuring the fraction of pixels whose depth agreement exceeds the confidence band derived from propagated uncertainties. If the inconsistency ratio for a candidate view exceeds the expected occlusion ratio (Eq. 6 in the MP-SfM paper), de-register the view and discard its associated structure.

Distinct from learned symmetry rejection (e.g., Doppelgangers, which is a per-pair learned classifier) — operates on already-reconstructed dense geometry and is less susceptible to domain shift. Distinct from [[sfm.registration-validation]], which typically uses track-count / reprojection heuristics rather than dense depth comparison.

Example fillers:
- [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] — the canonical instance; runs the check per newly registered view and once on the final reconstruction.
