---
title: Monocular Depth Estimation
type: thread
tags: [depth-estimation, monocular, metric-depth, relative-depth, foundation-model]
created: 2026-04-12
updated: 2026-04-12
sources: [papers/pataki2025_mp-sfm.md, papers/zhong2026_instantsfm.md, papers/li2025_megasam.md, papers/tang2025_dronesplat.md, papers/kim2025_multiview-geometric-gs.md, papers/chebbi2025_multiview-dense-matching.md]
status: draft
---

## Working hypothesis

Monocular depth has become a **load-bearing prior** for both classical and
learned 3D pipelines. It's no longer a standalone task — its value is measured
by how much it improves downstream SfM, 3DGS, or MVS. The current frontier is
integrating mono depth without over-constraining the geometry.

## Evidence

### As SfM prior
- [MP-SfM (Pataki 2025)](../papers/pataki2025_mp-sfm.md): uses [[Metric3Dv2]]
  depth + normals to eliminate the three-view overlap requirement. Mono depth
  makes SfM work on sparse imagery where COLMAP fails.
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

## Open questions
- Is metric monocular depth accurate enough to replace SfM depth maps for
  casual capture? MP-SfM says yes for sparse views; unclear at scale.
- Do diffusion-based depth models (MariGold) offer real advantages over
  discriminative ones (Depth Anything) for downstream reconstruction?
  No paper in this batch directly compares them in a reconstruction pipeline.
- What's the right way to fuse mono depth priors into multi-view optimization
  without over-constraining the geometry? InstantSfM uses depth-constrained
  Jacobians; MP-SfM uses depth/normal as additional residuals. No consensus.
- How does mono depth interact with feed-forward methods (DUSt3R predicts its
  own depth — does adding mono depth priors help or conflict)?

## Related threads
- [[feed-forward-structure-from-motion]] — mono depth as prior for SfM
- [[radiance-field-evolution]] — mono depth for sparse-view 3DGS
- [[gaussian-to-mesh-pipelines]] — depth supervision improves mesh quality

