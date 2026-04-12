---
title: "RADIOv2.5: Improved Baselines for Agglomerative Vision Foundation Models"
type: paper
tags: [vision-foundation-model, knowledge-distillation, multi-teacher, vision-transformer, token-merging, multi-resolution, VLM]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/fundamentals/heinrich_2025_radiov25.pdf
url: https://arxiv.org/abs/2412.07679
status: draft
---

📄 [Full paper](../../papers/fundamentals/heinrich_2025_radiov25.pdf) · [arXiv](https://arxiv.org/abs/2412.07679)

## TL;DR

RADIOv2.5 is an improved agglomerative vision foundation model that distills knowledge from multiple teacher models (DFN CLIP, SigLIP, DINOv2, SAM) into a single student ViT backbone. The paper identifies and fixes critical issues in prior agglomerative models: resolution mode switching (DINO-like features at low resolution, SAM-like at high), teacher imbalance, and excessive output tokens. Key solutions include multi-resolution training, mosaic augmentation, PHI-S feature normalization, replacing OpenAI CLIP with SigLIP for better VLM performance, and a token merging technique (ToMe) to maintain high-resolution information within a fixed token budget.

## Problem

Agglomerative models like AM-RADIO train a single vision encoder by distilling from multiple specialized teachers, but face several challenges: (1) a "mode switch" phenomenon where feature distributions shift significantly based on input resolution (low-res yields DINO-like features, high-res yields SAM-like features), (2) teacher distribution imbalance complicating loss weighting, (3) high-resolution processing generates too many tokens for downstream VLMs (quadratic attention complexity), and (4) standard local pooling (pixel unshuffle) loses fine-grained details when compressing tokens.

## Method

1. **Multi-resolution training**: Instead of training different teachers at different resolutions, train the student against all teachers at multiple resolutions simultaneously. This eliminates the mode switch by ensuring the student sees each teacher's signal at every resolution. Achieves consistent feature distributions across resolutions.

2. **Teacher updates**: Replace OpenAI CLIP with SigLIP 400m for better VLM downstream performance. Use DINOv2-g-reg and SAM-H as remaining teachers. Apply PHI-S normalization to balance teacher distributions.

3. **Token merging (ToMe)**: Apply bipartite matching-based token merging to compress $H \times W$ vision tokens down to a fixed count while preserving fine-grained information. Unlike pixel unshuffle which loses details through local pooling, ToMe performs global merging of similar tokens, retaining informative tokens and merging redundant ones. RADIOv2.5 benefits disproportionately from this technique compared to other encoders.

4. **Mosaic augmentation**: 2x2 and 4x4 mosaic training augmentations to improve robustness.

5. **Scale variants**: Released at multiple scales: ViT-B, ViT-L, ViT-H, and ViT-g.

Training uses DataComp-1B dataset, batch size 1024+128, 600k iterations with MSE feature distillation loss and cosine summary loss. Backbone pre-trained on ImageNet-1k.

## Results

- **ImageNet-1k**: Zero-shot 82.51% (ViT-H), k-NN 85.81% (ViT-H), up from 78.35% / 84.01% baseline (ViT-L)
- **Segmentation**: ADE20k mIoU 53.97% (ViT-H), up from 50.03% baseline
- **3D understanding (Probe3D)**: Depth 85.7%, Surface Normals 62.5% (ViT-H)
- **VLM benchmarks (with VILA)**: TextVQA 69.74% (with ToMe), DocVQA 52.33%, OCRBench 42.90% -- massive gains over baseline (57.01%, 24.19%, 25.60%)
- **SAM COCO instance segmentation**: 76.14% (ViT-H)
- **Multi-resolution robustness**: Scale equivariance variance dramatically reduced compared to AM-RADIO baseline
- Each incremental change (multi-res, SigLIP, ViT-H, ToMe) leads to improved metrics across the board

## Why it matters

RADIOv2.5 addresses the practical challenge of building a single vision encoder that works well across diverse downstream tasks -- from classification and segmentation to 3D understanding and vision-language reasoning. The multi-resolution training and token merging contributions are particularly important for deploying foundation models in real-world applications where input resolutions vary and computational budgets are constrained. The work establishes improved baselines that benefit the entire ecosystem of vision foundation model research.

## Relation to prior work

- Builds on AM-RADIO [Ranzinger et al.] as the baseline agglomerative model
- Distills from [[DINOv2]], [[SAM]], [[CLIP]] (DFN CLIP and [[SigLIP]])
- Relates to SAM-CLIP, UNIC, Theia, and PHI-S in the agglomerative VFM space
- Token merging technique from ToMeSD [Bolya et al.]
- Evaluated with [[VILA]] for VLM benchmarks
- Uses [[vision-transformer]] (ViT) backbone with CPE for multi-resolution support
- Multi-task evaluation via Probe3D and MLoRE protocols
- Eagle uses mixture of vision encoders at inference; RADIOv2.5 unifies at training

## Open questions / limitations

- Token merging adds inference overhead; the trade-off between compression ratio and information loss needs task-specific tuning
- Agglomerative approach is fundamentally limited by the quality of teacher models
- Training cost is substantial (600k iterations on DataComp-1B with multiple teachers)
- The optimal number and selection of teachers for different downstream tasks is not fully explored
- Feature normalization (PHI-S) helps but doesn't fully resolve the challenge of balancing disparate teacher distributions

## References added to the wiki

- [[RADIOv2.5]]
- [[AM-RADIO]]
- [[agglomerative-VFM]]
- [[token-merging]]
- [[DINOv2]]
- [[SAM]]
- [[SigLIP]]
- [[vision-transformer]]
