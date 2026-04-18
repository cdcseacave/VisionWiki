---
title: SfM registration validation
type: stage
slug: sfm.registration-validation
consumes: [proposed-registration, existing-reconstruction-depths]
produces: [validation-verdict-with-consistency-score]
invariants: [symmetry-ambiguities-detected]
provides_properties: [ghost-camera-detection]
requires_upstream_properties: [registration-proposed-by-upstream-stage]
data_regime: [incremental-sfm]
tags: [sfm, validation, forward-backward-depth, mp-sfm]
created: 2026-04-18
updated: 2026-04-18
---

Validates that a proposed camera registration is correct (not a symmetric / duplicate-view placement). Forward-backward depth consistency is the MP-SfM mechanism. Example fillers: [[mono-depth-normal-constrained-incremental-sfm_pataki2025]].
