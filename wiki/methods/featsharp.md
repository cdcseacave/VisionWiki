---
title: "FeatSharp: Your Vision Model Features, Sharper"
type: method
tags: [feature-upsampling, vit, super-resolution, distillation-preprocessing]
created: 2026-04-22
updated: 2026-04-22
sources: [wiki/papers/ranzinger2026_c-radiov4.md]
url: https://arxiv.org/abs/2502.16025
status: stub
---

## What it is

FeatSharp (Ranzinger et al. 2025, ICML) is a feature-upsampling method for
fixed-resolution vision backbones. Given a ViT teacher that operates at a
fixed input size (e.g. SigLIP2 at 384×384, OpenAI CLIP at 224×224), FeatSharp
produces higher-resolution feature maps (e.g. 1152×1152) that are compatible
with the backbone's original feature space. Used as preprocessing to align
fixed-resolution teachers with multi-resolution students in distillation.

FeatSharp is also reported to identify and suppress fixed-pattern noise
(ViT register tokens, ViTDet window-border artifacts) — the same class of
artifact that [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4's]]
shift-equivariant loss addresses on the student side.

## Role in the RADIO family

- [[cradiov4-agglomerative-distillation_ranzinger2026|C-RADIOv4]] uses FeatSharp
  to 3× upsample SigLIP2's 384px outputs to 1152px, so the high-resolution
  student can be distilled against semantically-meaningful teacher features
  at its target resolution. Without FeatSharp, the student would learn to
  mimic a low-resolution bottleneck when trained at high resolution.
- The C-RADIOv2-VLM model powering Llama Nemotron Nano VL 8B (currently #1 on
  OCR Bench v2) was trained with FeatSharp in the pipeline.

## To flesh out

- Exact upsampling mechanism (learned upsampler, joint-bilateral, etc).
- How the method preserves the original backbone's feature space (no new
  feature decoder needed).
- Ablation of FeatSharp-upsampled vs bilinear-upsampled teacher features in
  C-RADIOv4 training.
