---
title: Feed-forward SfM prior injection
type: stage
slug: feed-forward-sfm.prior-injection
consumes: [image-features, optional-intrinsics-poses-depth]
produces: [prior-conditioned-backbone-features]
invariants: [missing-priors-handled-gracefully]
provides_properties: [single-model-all-prior-subsets]
requires_upstream_properties: [per-block-mlp-injection-feasible]
data_regime: [dust3r-family]
tags: [pow3r, conditioning, dust3r]
created: 2026-04-18
updated: 2026-04-18
---

Per-block MLPs inject optional auxiliary priors (intrinsics, poses, sparse depth) into a DUSt3R-style backbone. Must bundle with modality-dropout training. Example fillers: [[pow3r-versatile-conditioning_jang2025]].
