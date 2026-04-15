---
title: Feed-Forward Structure from Motion
type: thread
tags: [sfm, pose-estimation, feed-forward, dust3r, mast3r, rommav2, transformer, test-time-training]
created: 2026-04-12
updated: 2026-04-12
sources: [papers/zhong2026_instantsfm.md, papers/pataki2025_mp-sfm.md, papers/yu2025_cusfm.md, papers/murai2025_mast3r-slam.md, papers/li2025_megasam.md, papers/zhao2025_diffusionsfm.md, papers/zhang2025_loger.md, papers/jin2026_zipmap.md, papers/zhang2024_cameras-as-rays.md, papers/jang2025_pow3r.md, papers/edstedt2025_roma-v2.md, papers/zhang2025_feed-forward-3d-survey.md, papers/chen2026_ttt3r.md]
status: draft
---

## Working hypothesis

The SfM landscape is splitting into three tiers:

1. **Accelerated classical** (InstantSfM, CuSfM): keep COLMAP's math, rewrite
   it for GPUs. 40x speedups, same accuracy. Low risk, high practical value.
2. **Hybrid classical + learned priors** (MP-SfM, MegaSaM, Pow3R): inject mono
   depth/normal priors into the classical pipeline. Extends SfM to harder cases
   (sparse views, dynamic scenes) while keeping optimization-based refinement.
3. **Fully feed-forward** (DiffusionSfM, LoGeR, ZipMap, MASt3R-SLAM): replace
   the pipeline with a single neural forward pass. Highest ceiling, least mature.

The DUSt3R/MASt3R family is the backbone of tier 3. The frontier is scaling
feed-forward methods to long sequences (hundreds/thousands of frames) without
quadratic attention costs.

## Evidence

### Tier 1: GPU-accelerated classical SfM
- [InstantSfM (Zhong 2026)](../papers/zhong2026_instantsfm.md): full global SfM
  in PyTorch. Depth-constrained Jacobians, GPU-native BA. 40x faster than
  [[colmap|COLMAP]] on 1K+ image datasets. The "just make COLMAP fast" approach.
- [CuSfM (Yu 2025)](../papers/yu2025_cusfm.md): CUDA SfM tailored for
  autonomous driving — multi-camera rig support, extrinsic refinement. Practical
  engineering over novelty.
- **Deep dive**: see [[gpu-native-sfm]] thread for the full Tier 1 story,
  including the shared playbook and InstantSfM-vs-CuSfM trade-offs.

### Tier 2: Classical + learned priors
- [MP-SfM (Pataki 2025)](../papers/pataki2025_mp-sfm.md): mono depth + normal
  priors from [[Metric3Dv2]] eliminate the three-view overlap requirement in
  incremental SfM. Key result: SfM works on sparse drone/indoor imagery where
  COLMAP fails.
- [MegaSaM (Li 2025)](../papers/li2025_megasam.md): learned motion segmentation
  + uncertainty-aware BA enables SfM from casual dynamic videos (people walking,
  cars moving). Bridges the "static scene assumption" gap.
- [Pow3R (Jang 2025)](../papers/jang2025_pow3r.md): lightweight conditioning
  injects optional camera intrinsics/poses/sparse depth into [[dust3r|DUSt3R]]. Shows
  that the DUSt3R architecture can absorb classical priors when available.

### Tier 3: Fully feed-forward
- [MASt3R-SLAM (Murai 2025)](../papers/murai2025_mast3r-slam.md): first
  real-time dense SLAM on [[mast3r|MASt3R]] priors. 15 FPS, no calibration needed.
  Proves the DUSt3R family can serve as a live SLAM backend.
- [DiffusionSfM (Zhao 2025)](../papers/zhao2025_diffusionsfm.md): diffusion
  model predicts ray origins + endpoints for joint structure and motion. Novel
  formulation — no explicit pose parameterization at all.
- [LoGeR (Zhang 2025)](../papers/zhang2025_loger.md): hybrid TTT + sliding
  window attention scales to 19K frames. Key innovation: [[test-time-training]]
  as a linear-time memory mechanism for long sequences.
- [ZipMap (Jin 2026)](../papers/jin2026_zipmap.md): bidirectional TTT fast
  weights, 20x faster than [[vggt|VGGT]]. Shows test-time-training is becoming the
  dominant paradigm for scaling feed-forward reconstruction.
- [TTT3R (Chen 2026)](../papers/chen2026_ttt3r.md): the *training-free*
  variant of the same idea. Reinterprets [[CUT3R]]'s recurrent state as a
  fast weight and derives a **closed-form confidence-guided learning rate**
  from the existing attention alignment — no retraining, no architecture
  change. 2× pose improvement over CUT3R on long sequences at 20 FPS / 6 GB
  for thousands of images. Distinguishes itself from LoGeR/ZipMap by being
  a plug-and-play patch to the recurrence rather than a new trained model,
  and articulates the unified `Tokenize → Update → Read → De-tokenize`
  formulation that separates full-attention (VGGT, Fast3R) from RNN
  (CUT3R, Point3R) pointmap models cleanly.
- [Cameras as Rays (Zhang 2024)](../papers/zhang2024_cameras-as-rays.md):
  Plucker ray-bundle camera representation + diffusion for sparse-view pose.
  Earlier work that planted the "cameras are just rays" idea.

### Cross-cutting: matching and features
- [RoMa v2 (Edstedt 2025)](../papers/edstedt2025_roma-v2.md): dense matcher
  with DINOv3 backbone and custom CUDA refinement. Even in feed-forward SfM,
  correspondence quality still matters — RoMa v2 is the current matching SOTA.
- [Survey (Zhang 2025)](../papers/zhang2025_feed-forward-3d-survey.md):
  comprehensive taxonomy confirming the three-tier split above. Identifies
  pointmap prediction (DUSt3R-style) as the dominant feed-forward paradigm.

## Emerging patterns

- **Test-time training** (LoGeR, ZipMap, TTT3R) is the scaling trick for feed-forward
  reconstruction: linear time, no quadratic KV cache, handles thousands of frames.
  Frontier question: does TTT need retraining (LoGeR, ZipMap) or can it be
  derived from the frozen model's own attention (TTT3R)? TTT3R's training-free
  closed-form learning rate is the cheapest answer yet.
- **DUSt3R/MASt3R** is the gravitational center — Pow3R extends it, MASt3R-SLAM
  runs it in real-time, DiffusionSfM rethinks it.
- **Monocular depth priors** are the bridge between classical and learned: MP-SfM
  and InstantSfM both use them to make classical BA more robust.

## Open questions
- Can feed-forward methods match COLMAP accuracy on large-scale outdoor scenes?
  InstantSfM matches it by being COLMAP-on-GPU; the fully learned methods
  (DiffusionSfM, ZipMap) haven't proven this yet.
- Is the endgame a single model (images → posed point cloud) or hybrid?
  Evidence leans hybrid — even LoGeR and ZipMap use DINOv2 backbones, not
  end-to-end training.
- How sensitive are feed-forward methods to domain shift? Most train on
  ScanNet/CO3D — outdoor generalization is under-tested.
- Will test-time-training replace attention for long-context 3D?

## Related threads
- [[radiance-field-evolution]]
- [[mono-depth-estimation]]
- [[gaussian-to-mesh-pipelines]] — VGG-T3 bridges feed-forward to mesh output

