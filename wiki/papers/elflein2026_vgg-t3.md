---
title: "VGG-T3: Offline Feed-Forward 3D Reconstruction at Scale"
type: paper
tags: [feed-forward-reconstruction, test-time-training, multi-view-stereo, linear-scaling, visual-localization]
created: 2026-04-12
updated: 2026-04-15
sources: []
local_paper: papers/mesh-reconstruction/elflein_2026_vgg-t3.pdf
url: https://arxiv.org/abs/2602.23361
status: draft
---

📄 [Full paper](../../papers/mesh-reconstruction/elflein_2026_vgg-t3.pdf) · [arXiv](https://arxiv.org/abs/2602.23361)

## TL;DR

VGG-T3 (Visual Geometry Grounded Test Time Training) is a feed-forward 3D reconstruction model that scales linearly with the number of input views by replacing the quadratic softmax attention in VGGT with a fixed-size MLP scene representation optimized via test-time training. It reconstructs 1,000-image collections in 54 seconds (11.6x faster than VGGT) while maintaining competitive reconstruction quality, and additionally enables feed-forward visual localization.

## Problem

Offline feed-forward multi-view reconstruction methods like VGGT, DUSt3R, and MASt3R achieve strong accuracy but their computational and memory requirements grow quadratically $O(n^2)$ with the number of input images, due to the variable-length Key-Value (KV) space in global self-attention layers. This makes them impractical for large image collections (hundreds to thousands of views). Existing linear-time alternatives (sliding window, block-sparse attention) sacrifice global scene consistency or remain asymptotically quadratic.

## Method

VGG-T3 applies two key ideas:

1. **KV Compression via Test-Time Training (TTT)**: Replaces the variable-length KV pairs in VGGT's global attention layers with fixed-size MLPs. At each global attention layer:
   - **Update step**: Input tokens are projected to Q, K, V; the KV information is compressed into a fixed-size MLP via test-time training (gradient-based optimization of MLP weights).
   - **Apply step**: Queries are passed through the frozen MLP to retrieve scene information, replacing the quadratic softmax attention with a linear operation.

2. **Post-Training Linearization**: Rather than training from scratch with TTT, the method fine-tunes a pre-trained VGGT model by replacing softmax attention with the TTT-based linear alternative. L2 normalization replaces LayerNorm to enable fast convergence from pre-trained weights.

The method outputs per-image depth maps, camera poses, and intrinsics. For visual localization, the optimized MLP is frozen and queried with new images to predict their camera poses relative to the reconstructed scene.

## Results

- **3D Reconstruction** (7scenes, DTU, ETH3D, NRGBD): Outperforms the other $O(n)$ baseline (TTT3R) on all benchmarks except marginally on 7scenes-D. Reduces Chamfer error by 2--2.5x on DTU, ETH3D, and NRGBD-D compared to TTT3R. Competitive with and sometimes surpasses $O(n^2)$ baselines on DTU.
- **Scaling**: Reconstructs 1k images in 54 seconds vs. VGGT's ~11 minutes (11.6x speedup). VGGT goes OOM beyond ~600 views.
- **Video Depth**: Substantially outperforms sequential $O(n)$ baselines on Bonn, KITTI, and Sintel; on par with $O(n^2)$ baselines.
- **Visual Localization**: Naturally supports feed-forward localization by querying the frozen MLP with novel views -- a capability not available in prior methods without separate pipelines.

## Why it matters

VGG-T3 addresses the fundamental scalability bottleneck of modern feed-forward 3D reconstruction. By showing that the KV scene representation can be compressed into fixed-size MLPs without significant quality loss, it opens the door to reconstructing large-scale scenes (landmarks, cities) from thousands of images in practical time. The unification of reconstruction and localization in a single model is also noteworthy.

## Pipeline contribution

- **KV compression via test-time-trained fixed-size MLP (N1)** — at each global-attention layer, Q/K/V projected, KV compressed into a fixed MLP via gradient-step TTT update, Qs passed through frozen MLP for retrieval. candidate thread: [[feed-forward-structure-from-motion]] Tier 3 · stage: *global attention* · replaces/augments: *VGGT's quadratic softmax attention* · expected gain: 11.6× speedup on 1K images; matches VGGT quality on DTU/ETH3D; 2–2.5× Chamfer reduction over TTT3R.
- **Post-training linearization from pre-trained VGGT (N2)** — L2 normalization replaces LayerNorm; fine-tune only the attention replacement. candidate thread: *feed-forward post-training* · stage: *architectural surgery on trained transformers* · expected gain: no from-scratch training needed.
- **Feed-forward visual localization via frozen updated MLP (N3)** — localize novel views by querying the scene MLP. candidate thread: *feed-forward localization* · stage: *output-time scene query* · expected gain: unified reconstruction + localization in one model.
- **Connection to [[gaussian-to-mesh-pipelines]] Paradigm D**: VGG-T3 outputs mesh/occupancy directly without per-scene optimization — natural bridge between feed-forward reconstruction and mesh extraction without a TSDF/marching-cubes post-pass.

## Relation to prior work

- Built on top of **VGGT** (Visual Geometry Grounded Transformer) as the base architecture.
- Uses **test-time training (TTT)** framework from Sun et al., extending it from language/video to multi-view 3D reconstruction.
- Compares against $O(n^2)$ methods: VGGT, FastVGGT, SparseVGGT, DUSt3R, MASt3R.
- Compares against $O(n)$ methods: TTT3R, Slam3R, VGGT-SLAM, VGGT-Long.
- Related to [[neural-implicit-surfaces]] via the MLP scene representation, though here the MLP encodes token-space KV mappings rather than geometry directly.
- Visual localization connects to Scene Coordinate Regression methods and PnP solvers.

## Open questions / limitations

- Small quality gap vs. $O(n^2)$ baselines that narrows with increasing number of images.
- The TTT optimization adds overhead per-layer; total speedup depends on collection size.
- Currently demonstrated for depth/pose prediction; extension to color/appearance reconstruction not explored.
- Post-training linearization requires the base VGGT model to be available and compatible.

## References added to the wiki

- [[[elflein2026_vgg-t3]]] (this page)
