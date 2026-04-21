---
title: Feed-forward SfM PTQ calibration sampling
type: stage
slug: feed-forward-sfm.ptq-calibration-sampling
consumes: [training-or-validation-image-sequences, trained-model-checkpoint]
produces: [calibration-samples, calibration-statistics]
invariants: [sample-distribution-covers-deployment-distribution, stability-across-random-seeds]
provides_properties: [stable-quantization-range-estimates, multi-view-distribution-coverage]
requires_upstream_properties: [converged-training-checkpoint, access-to-representative-images]
data_regime: [post-training-transform, calibration-only-no-gradient]
tags: [quantization-calibration, vggt, ptq, efficiency]
created: 2026-04-21
updated: 2026-04-21
---

Companion stage to `[[feed-forward-sfm.numerical-precision]]`: selects the calibration samples used during post-training quantization to estimate per-layer activation ranges. Separated as a first-class stage because in VGGT-class models, calibration-sample selection is unstable — the multi-view nature of the input distribution means naive random sampling yields per-run variance that dominates the PTQ error budget.

Fillers here address two failure modes unique to VGGT-family calibration:

- **Noisy extremes**: rare input distributions (e.g., textureless walls, extreme viewpoints) produce outlier activations that contaminate the range estimate. Deep-layer activation statistics can filter these samples before they enter the calibration pool.
- **Single-frame clustering misses multi-view structure**: clustering calibration candidates at image level ignores that VGGT's inductive bias operates at the frame *group* level. Frame-aware clustering groups samples that share multi-view distributional properties.

Example filler: [[frame-aware-ptq-calibration-sampling_feng2025]] (deep-layer statistics for noise filtering + frame-aware clustering for diversity).

Orthogonal to the quantization *transform* (`[[feed-forward-sfm.numerical-precision]]`) but practically `co_requires:` — PTQ paper headline numbers typically need matched calibration and transform to hit their accuracy floor.
