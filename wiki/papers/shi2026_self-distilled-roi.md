---
title: "Catching the Details: Self-Distilled RoI Predictors for Fine-Grained MLLM Perception"
type: paper
tags: [multimodal-llm, region-of-interest, self-distillation, fine-grained-perception, visual-question-answering, high-resolution]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/fundamentals/shi_2026_self-distilled-roi.pdf
url: https://arxiv.org/abs/2509.16944
status: draft
---

📄 [Full paper](../../papers/fundamentals/shi_2026_self-distilled-roi.pdf) · [arXiv](https://arxiv.org/abs/2509.16944)

## TL;DR

SD-RPN introduces a self-distilled Region Proposal Network for Multimodal Large Language Models (MLLMs) that identifies Regions of Interest (RoIs) in low-resolution images and selectively upscales only those regions. The RPN is trained using pseudo-labels derived from denoised internal attention maps of the MLLM itself, requiring no external annotations. Despite training on only 10K QA pairs, SD-RPN achieves over 10% absolute accuracy improvement on unseen benchmarks (TextVQA, DocVQA, V-Star) and operates efficiently via a single partial forward pass through middle LLM layers.

## Problem

MLLMs require high-resolution visual information for fine-grained perception (reading text, identifying small objects), but processing entire high-resolution images is computationally prohibitive. Existing RoI approaches face a trade-off: training-based methods require large-scale annotated datasets, while training-free methods that use the model's internal attention are computationally inefficient (requiring multi-pass prefill or slow auto-regressive decoding) and less accurate due to noisy attention maps.

## Method

1. **Pseudo-label generation** from noisy attention maps:
   - Extract response-to-image cross-attention maps $M_{RoI}$ from the MLLM
   - **Denoise** by removing sink tokens (identified via high L2-norm of feature vectors exceeding threshold $\tau_{norm}$)
   - **Label assignment**: Selective binary classification assigns foreground ($a_j \geq \tau_{fg} a_{max}$) and background ($j \notin B_{fg}$ and $a_j \leq \tau_{bg} a_{max}$), with ambiguous tokens ignored. A minimal bounding box $B_{fg}$ around foreground tokens prevents erroneous background labels on inactive parts of the true object.

2. **Self-Distilled RPN architecture**:
   - $R$ transformer blocks initialized from MLLM layers $B$ to $B+R$, built on top of $B$ frozen backbone layers
   - Predicts dense RoI map via $\hat{M}_{RoI} = Q_{RoI} K_v^T$ using query vectors from final user question tokens and visual token keys
   - Requires only a single partial forward pass (up to middle layers), completely decoupled from auto-regressive generation
   - Trained with BCE loss against pseudo-labels

3. **Two-stage inference**:
   - Stage 1: Predict RoI map, smooth with Gaussian filter, binarize to get mask $B$
   - Stage 2: Crop sub-images from identified regions, encode at high resolution
   - Two upscaling strategies: Box Upscaling (per-region crops) and S2 Upscaling (multi-scale)

Key insight: Using the MLLM's own responses (rather than stronger teacher responses) for pseudo-labels yields better results due to representational consistency.

## Results

- **LLaVA-1.5-7B + SD-RPN (10K training samples)**: +10.4% on TextVQA (47.8 to 58.2), +15.7% on DocVQA (23.2 to 38.9), +11.1% on V-Star (48.5 to 59.6)
- **Throughput**: 2.5x faster than ViCrop, 1.4x faster than S2 baseline at comparable or better accuracy
- **Data efficiency**: Performance saturates at ~10K training samples; 1K samples already show significant gains
- **Generalization across architectures**: Consistent gains when applied to DeepSeek-VL (+8.5% TextVQA) and Qwen2.5-VL (+2.4% TextVQA, +1.8% DocVQA)
- **Qwen2.5-VL + SD-RPN**: 83.5% TextVQA, 94.4% DocVQA -- competitive with much heavier approaches

## Why it matters

SD-RPN demonstrates that MLLMs contain sufficient internal localization knowledge to bootstrap their own fine-grained perception improvement without expensive annotations or large-scale retraining. The approach is model-agnostic, data-efficient, and computationally lightweight, making it a practical plug-in module for enhancing any MLLM's visual detail perception. This has direct implications for document understanding, OCR-dependent tasks, and visual grounding.

## Relation to prior work

- Integrates into [[LLaVA]]-1.5, [[DeepSeek-VL]], and [[Qwen2.5-VL]] architectures
- Competes with VILA-HD (training-heavy RoI) and ViCrop (training-free attention-based RoI)
- Uses [[clip|CLIP]] as the vision encoder within the MLLM pipeline
- Self-distillation paradigm relates to [[dinov2|DINOv2]] and other self-supervised VFM approaches
- Attention sink token phenomenon studied in [[dinov2|DINOv2]] register tokens (Darcet et al., 2023)
- S2 multi-scale strategy from Shi et al. (2024) used as one upscaling option

## Open questions / limitations

- Three failure modes identified: incomplete activation (partial RoI), localization errors (wrong region), and spatial relationship errors (correct objects but wrong reasoning about their arrangement)
- Dependent on the base MLLM's attention quality; weaker base models yield noisier pseudo-labels
- Currently processes RoI regions independently -- spatial relationships between multiple RoIs are not preserved
- Fixed threshold parameters ($\tau_{fg}$, $\tau_{bg}$, $\tau_{norm}$) may need tuning across different MLLM architectures

## References added to the wiki

- [[SD-RPN]]
- [[self-distillation]]
- [[region-of-interest]]
- [[multimodal-LLM]]
- [[LLaVA]]
- [[attention-sink-tokens]]
