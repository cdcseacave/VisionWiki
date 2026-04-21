---
title: Monocular Depth Estimation
type: thread
tags: [depth-estimation, monocular, metric-depth, relative-depth, foundation-model]
created: 2026-04-12
updated: 2026-04-21
sources: [papers/pataki2025_mp-sfm.md, papers/zhong2026_instantsfm.md, papers/li2025_megasam.md, papers/tang2025_dronesplat.md, papers/kim2025_multiview-geometric-gs.md, papers/chebbi2025_multiview-dense-matching.md, papers/yu2025_madpose.md, papers/wang2025_moge.md]
operating_points: [op:default]
status: draft
---

## Goal

Deliver monocular depth that actually improves downstream 3D pipelines — SfM, 3DGS, MVS, feed-forward reconstruction — without over-constraining the geometry. The thread tracks mono depth as a *load-bearing prior*, not a standalone task: success is measured by the downstream metric gain when the prior is injected, not by the depth's own absolute error.

## Goal contract (optional, structured)

```yaml
metric: [downstream-pose-AUC-gain, downstream-chamfer-reduction, prior-injection-robustness]
target_regime: [single-image, posed | unposed, static | dynamic, metric-scale | relative-scale]
constraints: [no-per-scene-training-on-depth, graceful-degradation-when-prior-wrong]
required_capabilities: [metric-depth, scale-consistency-across-frames, uncertainty-estimate]
# to be filled: preferred benchmark suite once the field coalesces
```

## Capability gaps

- **Calibrated per-pixel uncertainty** from mono depth — would let classical BA downweight uncertain regions automatically. Partial answer now available: [[depth-proportional-uncertainty-fusion_pataki2025]] provides a `drop-in` recipe (pixel-wise max of depth-proportional and model-predicted uncertainty, scaled by a single calibration constant) that works across Metric3Dv2, DepthPro, DepthAnything-v2. Remaining gap: diffusion-depth variance outputs (MariGold-family) — is their epistemic uncertainty better calibrated or still needs the same fusion?
- **Principled fusion of mono depth + mono normal + sparse-point priors** into one residual — no consensus. Search target: papers that ablate prior-combination strategies on the same benchmark.
- **Mono depth that composes with DUSt3R-style feed-forward depth** without redundancy. Search target: papers comparing mono-depth-as-init vs. DUSt3R-depth-as-init in downstream 3DGS.
- **Cross-pair / cross-frame shift consistency** — [[yu2025_madpose|MADPose]] solves per-pair affine corrections `(α, β₁, β₂)` but these don't propagate to a globally consistent per-image `β`. Would unlock: depth-aware pair-pose as SfM frontend (cross-ref [[gpu-native-sfm]] Bet #016, [[relative-pose-estimation]] Bet #016). Search target: 2026 papers solving global affine consistency of mono-depth across sequences.
- **Metric depth that is actually metric** — MADPose Fig. 6 shows fitted `β > 10%` of median depth even for metric-trained DA. Open whether this is a systematic bias of current metric MDE models or a property of the SfM-derived GT used for calibration. Search target: papers training metric MDE with ablation against real-world scaled LiDAR ground truth (no SfM-derived scale).

## Working hypothesis

Monocular depth has become a **load-bearing prior** for both classical and
learned 3D pipelines. It's no longer a standalone task — its value is measured
by how much it improves downstream SfM, 3DGS, or MVS. The current frontier is
integrating mono depth without over-constraining the geometry.

## Evidence

### As SfM prior
- [MP-SfM (Pataki 2025)](../papers/pataki2025_mp-sfm.md): uses [[Metric3Dv2]]
  depth + normals to eliminate the three-view overlap requirement. Mono depth
  makes SfM work on sparse imagery where COLMAP fails. Also contributes a
  **reusable uncertainty-calibration recipe**:
  [[depth-proportional-uncertainty-fusion_pataki2025]] — applicable to any
  downstream consumer of off-the-shelf mono-depth uncertainty.
- [InstantSfM (Zhong 2026)](../papers/zhong2026_instantsfm.md): depth-constrained
  Jacobians from mono depth estimates stabilize GPU-native bundle adjustment.

### As 3DGS initialization/regularization
- [DroneSplat (Tang 2025)](../papers/tang2025_dronesplat.md): MVS-guided
  voxel optimization uses depth for geometry, but mono depth could substitute
  in sparse-view drone capture.
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md): external MVS depth
  supervision → SOTA Gaussian surface quality. Shows depth priors (multi-view
  here, but mono depth is the single-image equivalent) consistently improve
  Gaussian geometry.

### As dynamic scene enabler
- [MegaSaM (Li 2025)](../papers/li2025_megasam.md): [[DepthAnything]] provides
  scale-consistent depth that enables SfM from casual dynamic videos. Mono depth
  is the glue that makes dynamic scene SfM tractable.

### In MVS pipelines
- [Chebbi 2025](../papers/chebbi2025_multiview-dense-matching.md): extends
  stereo similarity learning to multi-view without retraining — adjacent to
  mono depth in that learned priors replace hand-crafted cost volumes.

### As relative-pose prior (new 2026-04-21)
- [MADPose (Yu 2025)](../papers/yu2025_madpose.md): first wiki paper
  consuming mono depth at the **two-view pose-estimation** stage rather than
  inside multi-view BA. Explicitly models per-view affine ambiguity
  `D̂ᵢ = aᵢDᵢ + bᵢ` and solves `(α, β₁, β₂, R, t, f)` jointly via new
  calibrated 3pt / shared-focal 4pt / two-focal 4pt solvers
  ([[affine-corrected-minimal-relative-pose-solvers_yu2025]]). Results:
  +15 AUC@20° from shift modeling alone over scale-only baselines; +22
  AUC@10° on ScanNet-1500 shared-focal over point-only PoseLib. Headline
  empirical finding: fitted `β` is routinely >10% of median depth even for
  metric-trained DA — shift modeling is load-bearing, not a corner case
  (Fig. 6 of [[yu2025_madpose]]). Full thread: [[relative-pose-estimation]].
- [MoGe (Wang 2025)](../papers/wang2025_moge.md): newest MDE foundation
  model; preferred by [[yu2025_madpose|MADPose]] for calibrated relative
  pose. Affine-invariant geometry supervision designed to isolate the
  scale + shift ambiguity so downstream consumers fit `α, β` cleanly.

## Open questions
- Is metric monocular depth accurate enough to replace SfM depth maps for
  casual capture? MP-SfM says yes for sparse views; unclear at scale.
- Do diffusion-based depth models (MariGold) offer real advantages over
  discriminative ones (Depth Anything) for downstream reconstruction?
  No paper in this batch directly compares them in a reconstruction pipeline.
- What's the right way to fuse mono depth priors into multi-view optimization
  without over-constraining the geometry? InstantSfM uses depth-constrained
  Jacobians; MP-SfM uses depth/normal as additional residuals with propagated
  covariance ([[bilateral-normal-integration-with-uncertainty_pataki2025]]).
  No consensus; Bet #010 in [[gpu-native-sfm]] proposes to combine them.
- How does mono depth interact with feed-forward methods (DUSt3R predicts its
  own depth — does adding mono depth priors help or conflict)?
- Does **shift modeling** generalize to depth consumers beyond pose?
  [[yu2025_madpose|MADPose]] shows shift is load-bearing at the pair-pose
  stage. Does the same `(α, β)` correction help 3DGS depth-regularization
  (currently mostly scale-only) or MVS cost-volume fusion? Unmeasured.

## Pass B note (2026-04-21 ingest)

MADPose is the first thread-level addition since this thread's creation that
consumes mono depth *outside* of multi-view BA. It opens a fourth evidence
category (relative pose), expands the set of consumer stages, and surfaces
the cross-pair shift-consistency gap that links this thread to
[[gpu-native-sfm]] Bet #016 and [[relative-pose-estimation]] Bet #014. No
bets filed directly in this thread — mono-depth is consumed by others, and
bets live where the consumption happens. No refuted or shelved bets this
pass.

## Related threads
- [[feed-forward-structure-from-motion]] — mono depth as prior for SfM
- [[relative-pose-estimation]] — mono depth as prior for two-view pose (new 2026-04-21)
- [[radiance-field-evolution]] — mono depth for sparse-view 3DGS
- [[gaussian-to-mesh-pipelines]] — depth supervision improves mesh quality

