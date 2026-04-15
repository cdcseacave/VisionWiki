---
title: "Trident: Harnessing Vision Foundation Models for High-Performance, Training-Free Open Vocabulary Segmentation"
type: paper
tags: [open-vocabulary-segmentation, training-free, CLIP, SAM, DINO, vision-foundation-model, semantic-segmentation]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/fundamentals/shi_2024_open-vocab-segmentation.pdf
url: https://arxiv.org/abs/2411.09219
status: draft
---

📄 [Full paper](../../papers/fundamentals/shi_2024_open-vocab-segmentation.pdf) · [arXiv](https://arxiv.org/abs/2411.09219)

## TL;DR

Trident is a training-free [[open-vocabulary-segmentation]] framework that synergistically combines three [[foundation-model|vision foundation models]]: [[clip|CLIP]] for semantic representation, [[dinov2|DINO]] for object-level spatial correlation, and [[sam|SAM]] for fine-grained global aggregation. The key innovation is a "Splice-then-Segment" paradigm that first splices CLIP feature maps from sub-images and then uses SAM's correlation matrix for global aggregation, overcoming the limited receptive field problem of previous "Segment-then-Splice" approaches. Trident achieves a 4.2% mIoU improvement over prior SOTA across eight benchmarks.

## Problem

CLIP excels at open-vocabulary recognition but suffers from spatial-invariant features and is constrained to low input resolutions, making it suboptimal for semantic segmentation. Previous adaptations use a sliding window "Segment-then-Splice" approach that segments sub-images independently and merges results. However, as source image resolution increases, each sub-image's receptive field covers a decreasing fraction of the image, degrading classification accuracy. On Pascal VOC, increasing resolution from 336 to 688 pixels causes mIoU to drop by 5.8-9.7%.

## Method

**Splice-then-Segment paradigm**:

1. **Feature extraction**: Divide high-resolution image into sub-images via sliding window. Extract features from each using CLIP's image encoder (up to penultimate layer), with DINO providing spatially covariant semantic correlation within each sub-image.

2. **Splice**: Concatenate sub-image feature maps into a single integral feature map $I_{feat}$.

3. **Global aggregation**: Construct an affinity matrix $A$ from SAM's encoder features that enables cross-window attention:
   - Compute cosine similarity matrix $C$ from SAM features
   - Extract attention weights $W$ from SAM's last encoder layer
   - Combine into an affinity matrix: $A = \frac{W + M}{\|W + M\|}$ where mask $M$ suppresses pairs with cosine similarity below threshold $\epsilon$
   - Apply: $\tilde{I}_{feat} = A \cdot I_{feat}$

4. **Segment**: Compute final segmentation via $S = \arg\max_c \cos(\tilde{I}_{feat}, T_{emd})$ using CLIP text embeddings.

**SAM Refinement**: Convert CLIP's coarse segmentation into point, box, and mask prompts for SAM's decoder:
- Point prompts: pixel with maximum confidence score per connected region
- Box prompts: minimal axis-aligned bounding box per region
- Mask prompts: scaled ($\alpha = 0.005$) confidence scores within binary mask
- Using all three prompt types simultaneously improves refinement quality

## Results

- **Training-free SOTA across 8 benchmarks** with CLIP ViT-B/16:
  - Pascal VOC: 92.6% mIoU (without background), 82.4% (with background)
  - Pascal Context (59): 36.1% mIoU
  - ADE20k: 17.1% mIoU
  - Cityscapes: 40.2% mIoU (+4% over previous SOTA)
  - COCO Object: 37.3% mIoU (+3% over previous SOTA)
- **With OpenCLIP ViT-H**: Average improvement of 4.2% mIoU across all benchmarks, +5% on Pascal VOC and Cityscapes
- **Outperforms training-based methods**: Beats CLIP-DINOiser by ~5% mIoU average
- **Competitive with weakly supervised methods** despite being fully training-free

## Why it matters

Trident demonstrates that combining complementary strengths of different vision foundation models can achieve strong segmentation without any training. The Splice-then-Segment paradigm solves a fundamental scalability issue of adapting CLIP for high-resolution segmentation. This training-free approach preserves the full generalization capability of the constituent models and eliminates annotation requirements, making it immediately applicable to any open-vocabulary scenario.

## Relation to prior work

- Builds on [[MaskCLIP]] which identified spatial invariance in CLIP and proposed attention modification
- Extends ProxyCLIP which uses [[dinov2|DINO]] for spatial correlation in sub-image processing
- Uses [[sam|SAM]] for both global aggregation (encoder features) and refinement (decoder)
- Uses [[clip|CLIP]] (ViT-B/16 and OpenCLIP ViT-H) for semantic features and text-image alignment
- Compares against training-based methods: SAM-CLIP, CLIP-DINOiser, SAN, CAT-Seg
- Contrasts with Segment-then-Splice methods: MaskCLIP, SCLIP, ProxyCLIP
- The spatial invariance problem in CLIP relates to findings in [[vision-transformer]] attention mechanisms

## Open questions / limitations

- SAM's features capture low-level visual similarity rather than semantic relationships, causing over-segmentation of objects with distinct sub-parts
- Computational cost of running three foundation models (CLIP + DINO + SAM) in inference
- Performance on COCO Stuff (28.3% mIoU) lags behind SAM-CLIP (31.5%) which uses extensive training
- The affinity matrix threshold $\epsilon$ is a sensitive hyperparameter
- Limited to static images; video segmentation would require temporal consistency

## References added to the wiki

- [[Trident]]
- [[open-vocabulary-segmentation]]
- [[clip|CLIP]]
- [[dinov2|DINO]]
- [[sam|SAM]]
- [[MaskCLIP]]
- [[splice-then-segment]]


## In the wiki
- Cited by [[open-vocabulary-segmentation]] and [[open-vocab-2d-composition]] threads as the canonical training-free CLIP+DINO+SAM composition.
- See [[lifting-foundation-models-to-3d]] for how this 2D pattern transfers to 3DGS.
