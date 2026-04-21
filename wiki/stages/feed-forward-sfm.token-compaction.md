---
title: Feed-forward SfM token compaction
type: stage
slug: feed-forward-sfm.token-compaction
consumes: [per-frame-tokens]
produces: [compacted-tokens]
invariants: [cross-view-geometric-information-preserved, reference-view-coverage-preserved]
provides_properties: [fewer-tokens-than-input, spatially-balanced-coverage]
requires_upstream_properties: [tokenizer-output]
data_regime: [multi-view-transformer, long-sequence-500-plus-frames]
tags: [vggt, efficiency, token-merging]
created: 2026-04-21
updated: 2026-04-21
---

Pre-attention stage that reduces token count before the global-attention pass in VGGT-family feed-forward pointmap models. Introduced as a first-class stage because long-sequence inference (500+ frames) is quadratic in token count at `[[feed-forward-sfm.global-attention]]` — reducing the token count at this stage amplifies any downstream attention-mechanism savings.

Fillers here preserve cross-view geometric correspondences while dropping / merging tokens that provide redundant information. The key invariants are:

- **Reference-view coverage preserved** — pointmap models (VGGT, π³) predict geometry with respect to a reference view; stripping those tokens breaks the output contract.
- **Spatial balance preserved** — pure salience-based dropping concentrates retained tokens on texture-rich regions and starves the attention of mid-region coverage.

Example filler: [[vggt-token-merge-3-part-partition_shen2025]] (reference-frame + top-salient + region-random partitioning, 90% merge ratio, training-free).

Orthogonal to `[[feed-forward-sfm.global-attention]]` fillers: token-compaction reduces the attention problem size; sparse-attention or TTT-based fillers reduce the per-problem cost. The two compose multiplicatively.
