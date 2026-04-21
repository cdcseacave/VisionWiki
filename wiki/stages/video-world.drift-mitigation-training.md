---
title: Video-world drift-mitigation training
type: stage
slug: video-world.drift-mitigation-training
consumes: [clean-ground-truth-training-video, history-current-chunk-pairing]
produces: [training-loss-robust-to-imperfect-history]
invariants: [training-time-only-does-not-modify-inference-graph]
provides_properties: [inference-robustness-to-autoregressive-error-accumulation]
requires_upstream_properties: [autoregressive-chunked-training-protocol, flow-matching-or-diffusion-schedule-available]
data_regime: [clean-gt-training-data, autoregressive-bi-directional-or-causal-dit]
tags: [video-generation, drift-mitigation, self-augmentation, self-forcing, training-strategy]
created: 2026-04-21
updated: 2026-04-21
---

A training-time stage that closes the **observation bias** gap in autoregressive video generation: during training the model conditions on clean ground-truth history, but at inference it must condition on its own (imperfect) outputs. Per-step errors — color shifts, blurring, mild distortions — go uncorrected and compound over autoregressive steps, degrading quality. Fillers at this stage expose the model to inference-like history distributions during training so it learns to *correct* drift rather than propagate it.

**Why it needs to exist.** Pure FramePack-style temporal context compression alleviates drift by extending the effective horizon and anchoring on the input image, but does not close the train-test discrepancy. A dedicated training-time mechanism is therefore needed — one that is specifically a *training-graph modification*, distinguishing it from inference-time tricks like KV-cache management.

**Valid fillers**:
- Simulate inference-time degradation by running the model forward on itself (Self-Forcing-family — causal only, expensive on bi-directional DiTs).
- Proxy the degradation distribution by noise injection on a clean latent, leveraging the flow/diffusion schedule as the source of "plausible corruption" (Lyra 2.0's self-augmentation).

**Example fillers**:
- `[[self-augmented-history-training_shen2026]]` — one-step noise injection on encoded history with probability `p_aug=0.7`, `t ~ U(0, 0.5)` in the flow-matching schedule (Lyra 2.0).

**Downstream compatibility**: the property `inference-robustness-to-autoregressive-error-accumulation` is consumed (via `requires_upstream_property`) by `[[video-world.distillation]]` — distilling a teacher that lacks this property yields a fragile student.
