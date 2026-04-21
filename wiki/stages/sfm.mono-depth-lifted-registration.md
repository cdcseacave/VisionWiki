---
title: SfM registration via mono-depth-lifted points
type: stage
slug: sfm.mono-depth-lifted-registration
consumes: [current-reconstruction, candidate-new-view, two-view-matches, monocular-depth-per-registered-view]
produces: [registered-pose-for-new-view, newly-lifted-scene-points]
invariants: [reconstruction-connectivity-maintained, up-to-scale-or-rescaled-per-image]
provides_properties: [registration-from-two-view-overlap-only, works-with-dense-matchers]
requires_upstream_properties: [per-image-metric-or-up-to-scale-depth-with-uncertainty]
data_regime: [incremental-sfm, low-overlap, low-parallax, unposed-images]
tags: [sfm, incremental, mono-depth, mp-sfm]
created: 2026-04-21
updated: 2026-04-21
---

Registration variant that lifts feature points in already-registered views to 3D via per-image monocular depth, forming 2D-3D matches between the candidate view's 2D keypoints and the lifted 3D points. Solves the pose via PnP. Enables registration from **two-view overlap only**, relaxing the classical three-view-track requirement — the central move that lets incremental SfM consume dense pairwise matchers (MASt3R, RoMa) and scale to low-overlap / low-parallax captures.

Distinct from [[sfm.next-view-registration]], which requires tracks across ≥3 registered images. Depth maps must be rescaled per-view (see A1's median-ratio scaling in [[mono-depth-normal-constrained-incremental-sfm_pataki2025]], Eq. 1) before lifting, otherwise up-to-scale priors introduce inconsistent lift magnitudes.

Example fillers:
- [[mono-depth-normal-constrained-incremental-sfm_pataki2025]] — the canonical instance.
