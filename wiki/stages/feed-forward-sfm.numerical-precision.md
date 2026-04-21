---
title: Feed-forward SfM numerical precision
type: stage
slug: feed-forward-sfm.numerical-precision
consumes: [trained-model-weights, activation-distributions]
produces: [quantized-model-weights, quantization-transforms]
invariants: [task-accuracy-retention-within-tolerance]
provides_properties: [reduced-bits-per-weight-or-activation, memory-reduction, hardware-speedup]
requires_upstream_properties: [converged-training-checkpoint]
data_regime: [post-training-transform, deployment-time]
tags: [quantization, vggt, ptq, efficiency]
created: 2026-04-21
updated: 2026-04-21
---

Post-training transform stage that reduces the numerical precision of a trained feed-forward backbone's weights and activations, trading tolerable accuracy loss for memory and real-hardware speedup. Orthogonal to attention-mechanism stages — quantization applies across all linear layers regardless of whether the attention is dense, sparse, or TTT-based.

This stage sits outside the training loop (filled at deployment time), so its fillers are transforms applied to a frozen checkpoint, not training-recipe changes. Typical fillers target activation quantization (INT8 / W4A4 / lower); the failure mode is heavy-tailed distributions caused by special tokens (camera, register) or outlier channels that blow the quantization range.

Required companion stage: `[[feed-forward-sfm.ptq-calibration-sampling]]` supplies the calibration data used to derive quantization parameters.

Example filler: [[vggt-dual-smoothed-quantization_feng2025]] (pre-global Hadamard rotation + post-local channel smoothing, `co_requires` frame-aware calibration).

Typical operating point gains for W4A4 on VGGT: ~3.7× memory reduction, ~2.5× real-hardware speedup, ≥98% accuracy retention.
