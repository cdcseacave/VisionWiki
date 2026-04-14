---
title: "ZipMap: Linear-Time Stateful 3D Reconstruction via Test-Time Training"
type: paper
tags: [3d-reconstruction, test-time-training, linear-time, feedforward, implicit-scene-representation, scalability]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/jin_2026_zipmap.pdf
url: https://arxiv.org/abs/2603.04385
status: draft
---

📄 [Full paper](../../papers/sfm-slam/jin_2026_zipmap.pdf) · [arXiv](https://arxiv.org/abs/2603.04385)

## TL;DR

ZipMap is a stateful feed-forward 3D reconstruction model that achieves linear-time bidirectional reconstruction by replacing global self-attention with Test-Time Training (TTT) layers that compress an entire image collection into compact fast-weight MLPs. It processes over 700 frames in under 10 seconds on a single H100 GPU (75 FPS)--more than 20x faster than [[vggt|VGGT]]--while matching or surpassing the accuracy of quadratic-time state-of-the-art methods. The adapted fast weights also serve as a queryable implicit scene representation for novel-view point map synthesis.

## Problem

State-of-the-art feed-forward 3D reconstruction models like [[vggt|VGGT]] and pi3 rely on global self-attention with quadratic computational cost in the number of input images, making them prohibitively expensive for large image collections. Sequential approaches (CUT3R, TTT3R) reduce cost to linear but sacrifice reconstruction quality due to error accumulation and lack of bidirectional context. No existing method achieves linear scaling while maintaining bidirectional reconstruction quality.

## Method

ZipMap's architecture interleaves local window attention with large-chunk TTT layers across 24 blocks:

1. **Local Window Attention**: Standard self-attention with rotary positional encoding within each view independently, capturing spatial relationships within frames.

2. **Large-Chunk TTT Layer**: The core mechanism. Fast-weight MLP ($f_W$) implemented as SwiGLU is adapted via a single gradient step per layer on a virtual key-value reconstruction objective:
   - Keys and values projected from input tokens
   - Muon optimizer with Newton-Schulz orthonormalization for stable updates
   - Per-token learning rates predicted by a linear layer
   - Updated fast weights are applied to query tokens, analogous to cross-attention but with O(N) complexity

3. **Prediction Heads**: Camera head (quaternion, translation, 2 intrinsics), DPT-style point head (local pointmaps), depth head (with learned confidence maps), and query head (for novel-view synthesis of RGB and depth).

4. **Streaming Extension**: Fast weights can be updated online one view at a time for sequential reconstruction.

5. **Implicit Scene Representation**: The same fast weights, once updated with input images, can be queried with target ray map tokens to synthesize point maps from novel viewpoints in constant time.

Training uses scale-invariant point loss, depth loss with learned confidence, camera loss, and optional query losses for novel-view synthesis. Three-stage training on 64 H100 GPUs using 29 publicly available datasets.

## Results

- **RealEstate10K**: Pose AUC@5/15/30 of 53.34/74.97/84.30 (O(N) best), approaching pi3's 63.10/80.31/87.40 (O(N^2)).
- **Co3Dv2**: AUC@5/15/30 of 62.46/81.64/88.76, comparable to VGGT (67.84/83.95/89.99) and far ahead of CUT3R/TTT3R.
- **ScanNet**: ATE 0.015, matching pi3 (0.013) and VGGT (0.015). RPE trans 0.013 vs. pi3 0.009.
- **DTU/ETH3D**: Point map accuracy comparable to pi3 and VGGT, substantially outperforming CUT3R and TTT3R.
- **7-Scenes/NRGBD**: Dense and sparse view geometry comparable to quadratic methods.
- **Video Depth (Sintel/Bonn/KITTI)**: Consistently outperforms O(N) baselines and matches VGGT.
- **Runtime**: 700 frames in <10 seconds on H100. Over 20x faster than VGGT, ~3x faster than CUT3R/TTT3R (which process frames sequentially with lower GPU utilization).
- **DL3DV long-sequence**: Maintains low ATE matching pi3/VGGT as N grows, while CUT3R/TTT3R degrade significantly.

## Why it matters

ZipMap provides the first demonstration that linear-time feed-forward reconstruction can match quadratic-time methods in quality. The TTT fast-weight mechanism creates a natural implicit scene representation as a byproduct of reconstruction, opening possibilities for real-time scene querying without explicit 3D representations. This positions TTT-based architectures as a viable replacement for global attention in large-scale 3D vision.

## Relation to prior work

- Architecture built on [[vggt|VGGT]]'s design (DINOv2 encoder, frame attention, prediction heads) with global attention replaced by LaCT TTT blocks.
- Directly competes with [[vggt|VGGT]] and pi3 (quadratic) and CUT3R, TTT3R, Point3R (linear).
- TTT mechanism based on LaCT (Zhang et al., 2025b) and the original TTT framework (Sun et al., 2024).
- Fast-weight concept connects to classical Schmidhuber fast-weight systems and modern linear attention.
- Prediction heads follow [[dust3r|DUSt3R]]/pi3 conventions for pointmaps and camera parameters.
- Novel-view synthesis capability relates to [[nerf|NeRF]] and [[3d-gaussian-splatting]] but uses TTT weights as the scene representation.

## Open questions / limitations

- Co3Dv2 and RealEstate10K pose accuracy still trails pi3, suggesting TTT compression may lose some fine-grained geometric detail.
- Streaming mode (sequential updates) not as thoroughly evaluated as bidirectional mode.
- Novel-view synthesis quality (query head) limited; primarily produces point maps rather than photorealistic renders.
- Trained on 29 datasets; generalization to highly out-of-distribution scenarios not extensively tested.
- TTT fast-weight capacity may saturate for very large scenes (thousands of frames), though not demonstrated as a failure mode.

## References added to the wiki

- [[vggt|VGGT]]
- [[dust3r|DUSt3R]]
- [[test-time-training]]
- [[DINOv2]]
- [[structure-from-motion]]
- [[bundle-adjustment]]
- [[colmap|COLMAP]]
- [[GLOMAP]]
- [[CUT3R]]
- [[pi3]]
- [[3d-gaussian-splatting]]
- [[nerf|NeRF]]
