---
title: "LoGeR: Long-Context Geometric Reconstruction with Hybrid Memory"
type: paper
tags: [3d-reconstruction, long-sequences, test-time-training, sliding-window-attention, feedforward, scalability]
created: 2026-04-12
updated: 2026-04-12
sources: []
local_paper: papers/sfm-slam/zhang_2025_loger.pdf
url: https://arxiv.org/abs/2603.03269
status: draft
---

📄 [Full paper](../../papers/sfm-slam/zhang_2025_loger.pdf) · [arXiv](https://arxiv.org/abs/2603.03269)

## TL;DR

LoGeR (Long-context Geometric Reconstruction) scales dense 3D reconstruction to extremely long video sequences (up to 19K frames, 11.5km trajectories) without post-optimization. It processes video streams in chunks using bidirectional attention within each chunk, while a novel hybrid memory module--combining parametric Test-Time Training (TTT) for compressed global context and non-parametric Sliding Window Attention (SWA) for lossless local context--maintains coherence across chunk boundaries. Trained on 128 frames, it generalizes to thousands.

## Problem

Feedforward geometric foundation models ([[DUSt3R]], [[VGGT]], pi3) achieve strong short-window reconstruction but face a "context wall" (quadratic attention complexity) and a "data wall" (trained on short-context bubbles of dozens to ~100 frames). Naive efficiency improvements like FastVGGT mitigate memory bottlenecks but fail to generalize to large-scale scenes. Recurrent approaches like CUT3R compress all context into lossy hidden states, sacrificing precision. No single memory strategy balances local detail, adjacent-chunk alignment, and global structural integrity.

## Method

LoGeR processes input video in chunks with four operations per network block:

1. **Per-Frame Attention**: Standard self-attention independently on each frame's spatial tokens.

2. **Sliding Window Attention (SWA)**: Applied at a subset of depths (4 layers), attends to tokens from both previous and current chunks ($C^{m-1} \cup C^m$). Provides lossless short-range information highway for precise adjacent alignment.

3. **Chunk-wise TTT Layer**: Maintains fast weights $W^m$ updated per chunk via gradient descent on a self-supervised key-value reconstruction objective. Apply step injects compressed historical context; update step stores current chunk information. Anchors global coordinate frame and prevents scale drift.

4. **Chunk-wise Bidirectional Attention**: Full bidirectional attention within each chunk for powerful geometric reasoning.

**LoGeR\*** variant adds feedforward SE(3) alignment between consecutive chunks using overlapping frames for extremely long sequences.

**Training**: Curriculum strategy progressively increases chunk density and context length (48 to 128 frames). Data mixture heavily weighted toward large-scale-scene datasets (TartanAirV2). Losses include scale-invariant local pointmap, affine-invariant relative pose, and global pointmap losses.

## Results

- **KITTI**: Average ATE 18.65m (LoGeR\*) vs. TTT3R 72.86m (74% reduction), CUT3R 91.62m, DROID-SLAM 100.28m, and even DPV-SLAM++ 25.75m (optimization-based). Surpasses strongest optimization-based method VGGT-Long (27.64m) by 32.5%.
- **VBR benchmark** (up to 19K frames, 11.5km): 30.8% improvement over prior feedforward methods. Maintains global scale consistency where baselines exhibit severe drift.
- **7-Scenes**: 69.2% relative improvement in Chamfer distance over prior work.
- **ScanNet**: 80.0% relative ATE improvement. **TUM-Dynamics**: 66.1% relative ATE improvement.
- Trained on 128-frame sequences, generalizes to 1K frames directly and up to 19K frames with periodic state resets.

## Why it matters

LoGeR breaks both the "context wall" and "data wall" that limit feedforward 3D reconstruction to bounded scenes. By demonstrating city-scale dense reconstruction without any optimization backend, it represents a significant step toward replacing classical SfM/SLAM pipelines with purely feedforward models. The hybrid memory architecture provides a principled framework for balancing local precision and global consistency at linear computational cost.

## Relation to prior work

- Built on pi3 (Wang et al., 2026) as the bidirectional geometry backbone, initialized from pi3 weights.
- Extends [[VGGT]] and pi3 to long sequences; directly competes with FastVGGT, InfiniteVGGT, and StreamVGGT inference-time variants.
- TTT layers based on Test-Time Training (Sun et al., 2024) and LaCT (Zhang et al., 2025b), adapted for geometric reconstruction.
- Contrasts with recurrent approaches: CUT3R (persistent state), TTT3R (confidence-based single-frame streaming), and Point3R (explicit memory).
- Outperforms optimization-based methods: [[DROID-SLAM]], DPV-SLAM++, VGGT-Long, VGGT-SLAM.
- Introduces VBR benchmark for evaluating long-context reconstruction at unprecedented scale.

## Open questions / limitations

- Periodic state resets required for very long sequences (>1K frames) to prevent TTT weight saturation; inter-chunk alignment must compensate.
- SWA only inserted at 4 layers; increasing SWA depth may improve local consistency at higher compute cost.
- Depends on pi3/VGGT-quality bidirectional backbone; quality of intra-chunk reconstruction bounded by base model.
- Dynamic scenes not explicitly addressed; relies on base model's robustness to scene changes.

## References added to the wiki

- [[VGGT]]
- [[DUSt3R]]
- [[DROID-SLAM]]
- [[test-time-training]]
- [[sliding-window-attention]]
- [[structure-from-motion]]
- [[bundle-adjustment]]
- [[pointmap-prediction]]
- [[pi3]]
- [[CUT3R]]
