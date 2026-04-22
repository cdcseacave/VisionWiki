---
title: "PHI-S (Distribution Balancing for Label-Free Multi-Teacher Distillation)"
type: method
tags: [knowledge-distillation, multi-teacher, normalization, loss-balancing, agglomerative-vfm]
created: 2026-04-22
updated: 2026-04-22
sources: [wiki/papers/heinrich2025_radiov25.md, wiki/papers/ranzinger2026_c-radiov4.md]
url: https://arxiv.org/abs/2410.01680
status: stub
---

## What it is

PHI-S (Ranzinger et al. 2024) is a normalization recipe for the dense spatial
features of each teacher in a multi-teacher distillation setup. Different
teacher backbones (CLIP, DINOv2, SAM) produce features at very different
magnitudes; without normalization, the teacher with largest activations
dominates the MSE loss and the student under-fits the others. PHI-S normalizes
each teacher's output to a standardized distribution before computing the
distillation loss, removing the need for per-teacher loss-weight
hyperparameter search.

## Role in the RADIO family

- [[heinrich2025_radiov25|RADIOv2.5]] adopted PHI-S to balance the
  three-teacher (CLIP + SigLIP + DINOv2 + SAM) loss, replacing hand-tuned
  per-teacher weights.
- [[ranzinger2026_c-radiov4|C-RADIOv4]] extends PHI-S from *spatial* features
  to *summary tokens* via the angle-normalized summary loss. The original
  PHI-S paper explicitly did not address summary features because they were
  cosine-matched (inherently magnitude-normalized) — but C-RADIOv4 identified
  that *angular dispersion* is what matters for summary tokens, not magnitude,
  and PHI-S's philosophy ("balance the loss by the teacher's own statistics
  before adding to the gradient") generalizes directly.

## To flesh out

- Exact normalization formula (Hadamard transformation mechanics).
- Comparison vs naive per-teacher loss weighting.
- Whether it depends on ground-truth labels (name says label-free; confirm).
- Evidence that balanced-loss removes hyperparameter search in practice.
