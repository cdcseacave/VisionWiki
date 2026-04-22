---
title: SigLIP 2
type: method
tags: [vision-language, contrastive-learning, sigmoid-loss, siglip, text-image]
created: 2026-04-22
updated: 2026-04-22
sources: [wiki/papers/ranzinger2026_c-radiov4.md]
status: stub
---

## What it is

SigLIP 2 is the successor to SigLIP — a text-image contrastive model using a
pairwise sigmoid loss instead of the softmax loss used by CLIP. Released as
the frontier commercial-friendly text-image encoder as of 2025–2026. Adopted
as the **semantic alignment teacher** in C-RADIOv4 (replacing DFN-CLIP and the
original SigLIP that RADIOv2.5 used) and as the text encoder in recent VLMs
including Qwen3-VL.

Common variant referenced in the wiki: **SigLIP2-g-384** (giant, 1164M params,
384px input) — a C-RADIOv4 teacher.

## Why it matters for this wiki

- **Teacher in agglomerative distillation**: [[cradiov4-agglomerative-distillation_ranzinger2026]] uses SigLIP2-g-384 as the semantic alignment teacher. Prior AM-RADIO / RADIOv2.5 used DFN-CLIP + SigLIP v1.
- **Angular-dispersion asymmetry**: SigLIP2 summary features have angular dispersion ~0.694, much lower than DINOv3-7B's 2.186 — this asymmetry is what motivated C-RADIOv4's angle-normalized summary loss.
- **Fixed input resolution of 384px** forces any multi-resolution student distilling from it to upsample teacher features (C-RADIOv4 uses [[featsharp]] for 3× upsampling).

## To flesh out

- Primary paper + arXiv link (not yet ingested into wiki).
- Architectural differences vs SigLIP v1 and CLIP.
- Native resolution scaling story.
- Comparison vs CLIP / DFN-CLIP on standard zero-shot benchmarks.
