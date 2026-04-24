---
title: "SG-NN: Sparse Generative Neural Networks for Self-Supervised Scene Completion"
type: paper
tags: [scene-completion, self-supervised, sparse-cnn, regression-style, 3d-scan-completion]
created: 2026-04-24
updated: 2026-04-24
sources: []
local_paper: ""
url: https://arxiv.org/abs/1912.00036
code: https://github.com/angeladai/sg-nn
license_paper: arxiv-nonexclusive
license_code: unknown
status: stub
---

📄 [arXiv](https://arxiv.org/abs/1912.00036) · [code](https://github.com/angeladai/sg-nn)

> [!stub] Stub created 2026-04-24 because [[meng2026_seen2scene]] explicitly extends SG-NN's *self-supervised completion paradigm* (train on partial → more-partial pairs from real scans, no complete GT) into the modern flow-matching/ControlNet regime. SG-NN is the canonical reference for partial-data scan completion.

## TL;DR

SG-NN (CVPR 2020, Dai et al.) introduced the first **sparse generative network** that can be trained for **self-supervised scene completion from partial real-world scans** — without complete ground-truth meshes. The key trick: simulate "more incomplete" inputs by removing depth frames from existing partial scans, and supervise the network to predict back the original (less partial) scan. This is the conceptual ancestor of the frame-drop self-supervision in [[controlnet-frozen-flow-self-supervised-completion_meng2026]].

SG-NN is **regression-style**: deterministic output, single best-guess geometry per input. This contrasts with [[meng2026_seen2scene]]'s *generative* completion (stochastic, sampleable, layout-conditional). On the [[meng2026_seen2scene]] benchmarks, SG-NN is the primary baseline and Seen2Scene wins on completion fidelity + diversity.

## Why it matters here

- The frame-drop pretext task in [[controlnet-frozen-flow-self-supervised-completion_meng2026]] is a direct lift of SG-NN's self-supervised data construction (cite [13] in Seen2Scene).
- SG-NN bounds the regression-style ceiling on this task; Seen2Scene's wins quantify the value of moving from regression to generative completion.

## Open questions / what to expand on full ingest

- The exact frame-drop schedule used by SG-NN (uniform vs structured) — relevant to evaluating whether [[controlnet-frozen-flow-self-supervised-completion_meng2026]]'s frame-drop schedule could be improved.
- How SG-NN's sparse-CNN architecture compares to fVDB / sparse-DiT used by Seen2Scene — possible cross-pollination.

> [!needs-source] Full ingest deferred. Add `local_paper:` and full method analysis when promoted.
