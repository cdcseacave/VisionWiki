---
title: "SpatialLM: Training Large Language Models for Structured Indoor Modeling"
type: paper
tags: [large-language-model, 3d-scene-understanding, point-cloud, layout-estimation, object-detection, indoor-modeling]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/mesh-reconstruction/mao_2025_spatiallm.pdf
url: https://arxiv.org/abs/2506.07491
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/mao_2025_spatiallm.pdf) · [arXiv](https://arxiv.org/abs/2506.07491)

## TL;DR

SpatialLM is a multimodal large language model fine-tuned from open-source LLMs (Qwen2.5-0.5B) to process 3D point cloud data and generate structured indoor scene descriptions -- including walls, doors, windows, and oriented 3D object bounding boxes -- as Python-like text scripts. Trained on a new large-scale synthetic dataset of 12,328 indoor scenes (54,778 rooms), it achieves state-of-the-art layout estimation on Structured3D and competitive 3D object detection on ScanNet.

## Problem

Structured indoor modeling -- extracting architectural layouts and object boxes from raw RGBD scans -- has relied on task-specific network architectures (e.g., RoomFormer, V-DETR) that cannot leverage the broad capabilities of pre-trained language models. While autoregressive approaches like SceneScript exist, they use specialized structured language commands and train Transformers from scratch, limiting generalization. No prior work had successfully fine-tuned general-purpose LLMs for 3D structured scene understanding, partly due to the lack of large-scale training data.

## Method

SpatialLM follows the standard multimodal LLM architecture:

1. **Point Cloud Tokenization**: Uses the Sonata point cloud encoder at 2.5cm finest resolution to convert 3D point clouds into token sequences that can be processed by the LLM.

2. **Structured Output as Python Scripts**: Scene descriptions are represented as Python-like scripts with predefined object categories, walls, doors, and windows. This leverages the LLM's built-in coding capability and is human-interpretable and easily extensible.

3. **Large-Scale Synthetic Dataset**: Collected 12,328 indoor scenes (54,778 rooms) with high-quality 3D annotations, enabling effective pre-training before fine-tuning on downstream benchmarks.

4. **Single-Stage Training**: Fine-tuned directly from Qwen2.5-0.5B with the point cloud encoder, using a single training stage on the synthetic dataset.

The auto-regressive nature allows generating plausible layouts even from partial observations.

## Results

- **Layout Estimation (Structured3D)**: After pre-training on the new dataset and fine-tuning on Structured3D, achieves IoU2D@0.25 of 94.3 and IoU2D@0.5 of 93.5, outperforming RoomFormer (83.4/81.4) and SceneScript (90.4/89.2).
- **3D Object Detection (ScanNet)**: F1 IoU3D@0.25 of 65.6 and IoU3D@0.5 of 52.6, competitive with V-DETR (65.1/56.8) and substantially outperforming SceneScript (49.1/36.8).
- **Zero-Shot on Video Reconstructions**: Demonstrates strong resilience to noisy point clouds from MASt3R-SLAM, predicting full object sizes and orientations from sparse/occluded inputs without fine-tuning.
- Training directly on small downstream datasets (ScanNet, Structured3D) alone yields poor results; the large synthetic pre-training dataset is essential.

## Why it matters

SpatialLM demonstrates a feasible path for using general-purpose LLMs in 3D spatial understanding tasks, replacing task-specific architectures with a single unified model. The Python-script output format is a clever design choice that leverages LLM coding capabilities while being human-readable and extensible. This work is a step toward foundation models that can understand, reason about, and interact with structured 3D scenes.

## Relation to prior work

- Builds on the multimodal LLM paradigm, using **Qwen2.5-0.5B** as the base model and **Sonata** as the point cloud encoder.
- Compares against **RoomFormer** (specialist layout model) and **SceneScript** (autoregressive structured language model) for layout estimation.
- Compares against **V-DETR** (specialist 3D detector based on DETR) for object detection.
- Uses point clouds from RGBD scans or reconstructed via MASt3R-SLAM from monocular video.
- Related to broader 3D understanding work: ScanQA, 3D-LLM, LEO, and other 3D multimodal models.

## Open questions / limitations

1. **Cross-domain generalization**: Significant differences between point clouds from monocular videos, RGBD scans, and LiDAR; fine-tuning on specific datasets still needed for best performance. Scaling data and model size may help.
2. **Effect on language capabilities**: Impact on the base LLM's natural language processing and reasoning abilities not evaluated.
3. **Closed vocabulary**: Uses predefined object categories, limiting the LLM's open-ended language capabilities. Future work includes open-vocabulary detection and 3D VQA.
4. **Small objects**: Difficulty with small objects like "picture" and "sink" categories in ScanNet.

## References added to the wiki

- [[mao2025_spatiallm]] (this page)
