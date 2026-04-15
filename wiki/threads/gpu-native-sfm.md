---
title: GPU-Native SfM (Tier 1 — Accelerated Classical)
type: thread
tags: [sfm, gpu-acceleration, bundle-adjustment, colmap, glomap, cuda, pytorch]
created: 2026-04-12
updated: 2026-04-15
sources: [papers/zhong2026_instantsfm.md, papers/yu2025_cusfm.md]
status: draft
---

## Working hypothesis

There's an under-hyped but practically dominant research direction in SfM:
**keep the classical math, move it to the GPU**. Unlike the feed-forward
revolution (DUSt3R/MASt3R/VGGT), this tier doesn't replace COLMAP — it
reimplements it 10×–40× faster while preserving the accuracy guarantees that
make classical optimization trusted in production.

Two papers in the current wiki illustrate the two ends of this spectrum:
[InstantSfM](../papers/zhong2026_instantsfm.md) (general-purpose, PyTorch-native)
and [CuSfM](../papers/yu2025_cusfm.md) (domain-specialized for autonomous
driving, CUDA+TensorRT+Ceres).

## The classical baseline being accelerated

Both Tier 1 papers measure themselves against the same target: Schönberger's
2016 COLMAP system. It's worth pinning down *what* they're reimplementing.

- [Schönberger & Frahm 2016 — SfM](../papers/schonberger2016_colmap-sfm.md):
  the incremental SfM pipeline as meticulously re-engineered open source.
  Multi-model RANSAC for geometric verification, expected-gain next-best-view
  selection, iterative BA + re-triangulation loop. **Strengths**: robust on
  internet photo collections, scales to tens of thousands of images, still
  the accuracy benchmark. **Weaknesses**: CPU-bound; quadratic-ish in image
  count; the BA solve is the bottleneck InstantSfM and CuSfM attack.
- [Schönberger et al. 2016 — MVS](../papers/schonberger2016_colmap-mvs.md):
  the companion pixelwise-view-selection PatchMatch MVS stage with joint
  photometric + geometric consistency. **Strengths**: still the accuracy
  reference for dense classical MVS a decade later. **Weaknesses**: slow
  enough that modern pipelines route around it (learned MVS, Gaussian
  Splatting) when speed matters.

The persistent gravity of these two 2016 papers is itself evidence for the
"keep the math, change the hardware" thesis: ten years of feed-forward
alternatives have not displaced COLMAP as the accuracy benchmark. Tier 1
papers exist because the math is still correct.

## The shared playbook

Despite very different designs, both papers follow the same four-step recipe:

1. **Keep COLMAP/GLOMAP math.** Bundle adjustment, factor graphs,
   Levenberg-Marquardt. The classical formulation wasn't broken, just slow.
2. **Move data-parallel work to the GPU.** The specific mechanism differs
   (PyTorch sparse ops vs. CUDA+TensorRT), but both target Jacobian assembly
   and the normal-equation solve.
3. **Inject learned priors as constraints, not replacements.** InstantSfM
   uses [[monocular-depth-estimation]]; CuSfM uses SLAM prior poses + learned
   ALIKED features. Neither trusts a neural net to output final poses.
4. **Deliver 10×–40× speedup with preserved or improved accuracy.**

## Evidence

### [InstantSfM (Zhong 2026)](../papers/zhong2026_instantsfm.md)
- **What it is**: a GPU reimplementation of global SfM math in pure PyTorch.
- **Key novelties**: depth-constrained Jacobian structure (metric depth enters
  both global positioning and BA through shared Jacobian columns, not as a
  post-hoc alignment); dynamic parameter extraction to prevent rank-deficient
  normal equations when outliers temporarily invalidate points/cameras.
- **Scope**: general-purpose, designed to *replace* [[colmap|COLMAP]]/[[glomap|GLOMAP]]
  for anyone building a learning pipeline (3DGS/NeRF/robotics).
- **Benchmarks won**: MipNeRF360 NVS, ScanNet/ScanNet++ Chamfer distance.
- **Speedup**: 1.5×–40× over COLMAP, up to 12× over GLOMAP (100–5000 images).

### [CuSfM (Yu 2025, NVIDIA)](../papers/yu2025_cusfm.md)
- **What it is**: a CUDA-accelerated SfM system tailored for autonomous
  driving, built around a factor graph optimization framework.
- **Key novelties**: non-redundant data association that exploits prior SLAM
  trajectories to build a minimal view graph; TensorRT-accelerated ALIKED
  features; two-stage factor graph optimization (robust-loss warmup →
  stringent-outlier final pass); multi-camera rig extrinsic refinement.
- **Scope**: domain-specialized. Requires prior trajectory (PyCuVSLAM /
  ORB-SLAM2); designed to *refine* SLAM output for ground-truth generation.
- **Benchmarks won**: KITTI ATE (0.040m vs COLMAP 0.406m, 90.1% improvement),
  NVIDIA SDG simulation.
- **Speedup**: ~6× total over COLMAP, 20× mapping-only on KITTI 100-frame runs.

## The trade-offs split

| Dimension | InstantSfM | CuSfM |
|-----------|-----------|-------|
| Target use case | Anyone doing learning-based 3D | Autonomous driving specifically |
| Implementation | Pure PyTorch | CUDA + TensorRT + Ceres |
| Input assumptions | Unordered images, optional depth priors | Sequential data + SLAM prior |
| Feature extraction | External (RootSIFT) | ALIKED via TensorRT |
| Multi-camera rigs | No | Yes — joint pose + extrinsic calibration |
| Can run from scratch? | Yes | No |
| Deep-learning integration | First-class (PyTorch-native) | Tooling-level only |
| Novelty axis | Mathematical (Jacobian formulation) | Systems-level (pipeline design) |

## Why this tier matters near-term

While Tier 3 (fully feed-forward — [[feed-forward-structure-from-motion]])
grabs the attention, Tier 1 is where production SfM is heading because:

- **Accuracy is proven**: same math as COLMAP, validated on decades of benchmarks.
- **Failure modes are understood**: outliers, degenerate configurations,
  rank-deficient solves — all well-studied.
- **Learned priors are additive**: when the mono depth prior is wrong, the
  system degrades gracefully rather than catastrophically.
- **Drop-in replacement**: InstantSfM specifically positions itself as a
  faster COLMAP — no pipeline rewrites needed.

## Open questions

- Does InstantSfM's PyTorch-native design enable end-to-end differentiation
  through SfM in a way that CUDA-backed systems can't? None of the 2026
  papers exploit this yet.
- How far can CuSfM's SLAM-prior approach generalize beyond driving? The
  idea — "refine a coarse trajectory" — applies to drone flights, handheld
  phone scans, AR/VR, but no paper has tested it there.
- What's the right way to fuse **multiple** learned priors (depth + normal +
  correspondence) into classical BA? MP-SfM uses depth+normal; InstantSfM
  uses depth only; CuSfM uses SLAM-poses+features. No consensus.
- Will Tier 1 survive as Tier 3 matures, or does the answer just become
  "feed-forward + fast BA refinement pass"?

## How this connects

- [[feed-forward-structure-from-motion]] — this thread is Tier 1 of that
  thread's three-tier taxonomy; the other two tiers (hybrid, fully learned)
  are covered there.
- [[mono-depth-estimation]] — mono depth is the dominant learned prior for
  Tier 1 (InstantSfM, MP-SfM, MegaSaM).
- [[radiance-field-evolution]] — InstantSfM specifically targets 3DGS/NeRF
  pipelines as its downstream consumer.

---

## Goal & success criteria

Produce a posed-3D reconstruction at COLMAP-grade metric accuracy (pose AUC, Chamfer, T&T F-score) at 10×–40× the throughput. "Better" = same or better accuracy *and* ≥10× wall-clock speedup *and* drop-in replacement for COLMAP in downstream pipelines (3DGS / NeRF / robotics). Calibration-grade poses are the non-negotiable floor — trade-offs of accuracy for speed are out of scope for this thread.

## Current SOTA pipeline (as of 2026-04-15)

Stage-by-stage for a general-purpose GPU-native SfM:

1. **Feature extraction**: external — RootSIFT (InstantSfM) or ALIKED via TensorRT (CuSfM). Paper: [zhong2026_instantsfm] / [yu2025_cusfm]. Open: does replacing SIFT with DINOv3-head matcher (RoMa v2 frontend) shift the speed/accuracy frontier?
2. **Matching + geometric verification**: multi-model RANSAC, scene-graph augmentation. Paper: [schonberger2016_colmap-sfm]'s N1 carried forward unchanged.
3. **Global positioning + depth-constrained Jacobian BA**: PyTorch GPU sparse ops. Paper: [zhong2026_instantsfm]. Or two-stage factor-graph (robust → stringent) for rig-with-prior-trajectory workloads. Paper: [yu2025_cusfm].
4. **Iterative triangulation + BA loop**: Paper: [schonberger2016_colmap-sfm]'s N3 — still required for final polish.

## Pipeline lineage

- 2016 · baseline: COLMAP CPU SfM. Driver: [schonberger2016_colmap-sfm] + [schonberger2016_colmap-mvs].
- 2024 · accuracy twin on GPU: GLOMAP replaces incremental loop with global rotation averaging at COLMAP accuracy. Driver: [pan2024_glomap].
- 2026 · general-purpose GPU: InstantSfM 1.5×–40× over COLMAP, 12× over GLOMAP via depth-constrained Jacobians. Driver: [zhong2026_instantsfm].
- 2025 · domain-specialized GPU: CuSfM 6× over COLMAP on driving with prior SLAM trajectory. Driver: [yu2025_cusfm].

## Candidate components / not yet integrated

- **Learned feature frontend** (RoMa v2 on DINOv3, LoFTR) in place of RootSIFT — appears in research matchers but not yet inside a GPU-native classical SfM. Blocked on: speed parity; no clean benchmark.
- **Pow3R conditioning** as an auxiliary prior into BA — [jang2025_pow3r] injects partial priors into DUSt3R; adapting to classical BA is untried.

## Open questions & synthesis bets

- Can InstantSfM's PyTorch-native design enable end-to-end differentiation through SfM? No 2026 paper exploits this yet. **Synthesis bet**: *backprop a rendering loss (3DGS PSNR) through InstantSfM poses — joint SfM+radiance-field training in one graph.* Mixes [zhong2026_instantsfm] + any 3DGS method.
- **Synthesis bet**: *CuSfM's SLAM-prior two-stage factor graph applied to drone / phone scans* — idea generalizes beyond driving; no paper tested outside cars.
- **Multi-prior fusion**: MP-SfM uses depth+normal; InstantSfM uses depth; CuSfM uses SLAM-poses+features. No consensus. **Synthesis bet**: *combine mono depth + mono normal + DINOv3 dense-matching confidence into a single Jacobian augmentation* — each prior informs a different residual axis.

## Contradictions & tensions

- Feed-forward pointmap methods ([vggt], [dust3r]) claim parity with COLMAP on some benchmarks but lag on calibration-grade outdoor metrics. Thread-position: GPU-native classical remains the reference; feed-forward is not yet a drop-in replacement for calibration workflows.

