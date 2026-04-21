---
title: Radiance Field Evolution
type: thread
tags: [nerf, 3dgs, differentiable-rendering, novel-view-synthesis, sparse-voxels]
created: 2026-04-11
updated: 2026-04-21
sources: [papers/park2023_camp.md, papers/xie2025_gauss-mi.md, papers/tang2025_dronesplat.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md, papers/kim2025_multiview-geometric-gs.md, papers/guo2025_ea-3dgs.md, papers/deng2026_vpgs-slam.md, papers/lin2024_vastgaussian.md, papers/barron2022_mip-nerf-360.md, papers/barron2023_zip-nerf.md, papers/wang2026_feed-forward-3d-scene-modeling.md, papers/shen2026_lyra2.md]
operating_points: [op:quality-per-scene, op:city-scale, op:neural-free]
status: draft
---

## Goal

Produce a radiance field from posed images that balances **rendering quality** (PSNR/SSIM/LPIPS), **rendering speed** (FPS at 1080p), **training cost** (per-scene wall-clock), and **scalability** (memory at city-scale). No single component dominates all four; the thread tracks per-operating-point winners.

## Goal contract (optional, structured)

```yaml
metric: [PSNR, SSIM, LPIPS, inference-FPS@1080p, train-time-min, peak-memory-GB]
target_regime: [posed-images, bounded | unbounded, object-or-room | city-scale, static-scene-default]
constraints: [per-scene-optimization-acceptable, gpu-available, commercial-license-preferred]
required_capabilities: [novel-view-synthesis, geometry-extraction-optional, appearance-decoupling]
```

## Working hypothesis

The field has moved decisively from implicit MLPs (NeRF) to explicit primitives
(3DGS), with the current frontier being: (a) how to get geometry quality to
match or exceed neural implicit surfaces, and (b) how to scale to large outdoor
scenes without blowing up memory. A third competing direction — sparse voxel
rasterization — sidesteps Gaussians entirely while matching their speed.

## Evidence

### The NeRF → 3DGS transition
- [Mip-NeRF 360 (Barron 2022)](../papers/barron2022_mip-nerf-360.md) is the
  last implicit-NeRF paper to materially move the frontier. Three coordinated
  tricks — a non-linear scene contraction for unbounded captures, a
  proposal-network-as-online-distillation for cheap importance sampling,
  and a first-principles **distortion regularizer** against floaters and
  background collapse — produce 57% lower MSE than Mip-NeRF, and the
  dataset it introduced became the canonical outdoor-NVS benchmark for
  every subsequent radiance-field paper.
- [Zip-NeRF (Barron 2023)](../papers/barron2023_zip-nerf.md) was the
  high-water-mark of implicit radiance fields: fuses Mip-NeRF 360's
  contraction + proposal network with Instant-NGP's hash-grid speed,
  achieving **8× lower error than Mip-NeRF 360** at ~1h training. Its
  existence made the NeRF→3DGS hand-off a fair fight — 3DGS won not on
  raw quality but on *edit-ability* and real-time rendering, not on a
  weak NeRF baseline.
- [CamP (Park 2023)](../papers/park2023_camp.md) showed that NeRF's joint
  pose-radiance optimization is brittle due to ill-conditioned camera
  parameterization. Camera preconditioning via whitening reduced RMSE by 67%
  (stacked on Zip-NeRF) — but this is a patch on an inherently fragile
  pipeline, and no 3DGS paper in this batch has bothered replicating the
  trick. That's a tell: when pose jitter matters enough to invent CamP, the
  field is already shopping for a more robust representation.
- The 3DGS papers in this batch (6 of 7 radiance-field papers) all build on
  [[3d-gaussian-splatting]] as the base representation, confirming its dominance.

### Quality frontier: geometry from Gaussians
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md): MVS depth + normal
  regularization pushes 3DGS to SOTA explicit surface reconstruction on DTU/TNT.
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md): per-Gaussian SDF values
  enable relightable inverse rendering — bridging appearance and geometry without
  separate neural fields.
- [DroneSplat (Tang 2025)](../papers/tang2025_dronesplat.md): MVS-guided voxel
  optimization + distractor masking makes 3DGS robust under drone imagery
  (sparse views, dynamic objects, lighting variation).

### Scaling: outdoor and large scenes
Three distinct attacks on the same problem — they are orthogonal and could be
stacked.

- **Spatial partitioning.** [VastGaussian (Lin 2024)](../papers/lin2024_vastgaussian.md)
  — the *first* real-time 3DGS method for aerial / city-scale scenes. Ground-plane
  `m × n` partition with airspace-aware visibility (floaters live off the
  surface, so visibility must cover the full vertical column, not just the
  surface convex hull). Per-cell parallel training + seamless merge matches or
  beats Mega-NeRF/Switch-NeRF on SSIM and LPIPS *every scene*, at 170+ FPS
  and 10× shorter training. Decoupled appearance modeling (per-image CNN
  transforming the rendered target, not the representation) is the
  rasterization-compatible successor to NeRF-W's GLO embeddings.
- **Compression.** [EA-3DGS (Guo 2025)](../papers/guo2025_ea-3dgs.md):
  tetrahedral mesh init + codebook quantization achieves 5.2x compression for
  outdoor scenes.
- **Incremental SLAM.** [VPGS-SLAM (Deng 2026)](../papers/deng2026_vpgs-slam.md):
  voxel-based progressive 3DGS SLAM handles large-scale indoor/outdoor with loop
  closure, avoiding a priori partitioning.

### Alternative representations
- [SVRaster (Sun 2025)](../papers/sun2025_sparse-voxels-rasterization.md):
  sparse voxel rasterization with Morton ordering matches 3DGS quality/speed
  *without Gaussians at all* — neural-free, artifact-free, and naturally
  produces meshes via [[marching-cubes]]. This may be a dark horse.

### Active reconstruction
- [GauSS-MI (Xie 2025)](../papers/xie2025_gauss-mi.md): real-time mutual
  information from 3DGS for next-best-view selection — showing 3DGS is mature
  enough to serve as a live reconstruction backend, not just offline.

## Open questions
- Which representation wins for *editable* geometry vs pure novel-view synthesis?
- ~~How does 3DGS composition interact with traditional bundle adjustment?~~
  Partially answered: CamP shows preconditioning helps NeRF; 3DGS papers
  mostly rely on COLMAP poses. Direct 3DGS+BA co-optimization still rare.
- What's the current best path from a Gaussian splat to a usable textured mesh?
  → See [[gaussian-to-mesh-pipelines]] — now has 7 papers addressing this.
- Is SVRaster's sparse voxel approach a serious competitor to Gaussians, or
  a niche alternative?
- ~~Can 3DGS scale to city-scale reconstruction without quantization/LOD tricks?~~
  Partially answered: VastGaussian scales to aerial UrbanScene3D captures via
  spatial partitioning alone (no quantization), beating NeRF-based large-scene
  methods on quality and speed. *Open*: true city-scale (km²) likely still needs
  partitioning + compression layered together; no automatic cell-count
  selection exists.
- Can the decoupled appearance-modeling trick (train-time 2D transform,
  discarded at inference) be ported to *other* rasterization-based renderers
  (SVRaster, neural primitives) as a general recipe against photometric drift?
- **4D Gaussian frontier.** [[wang2026_feed-forward-3d-scene-modeling]] §4.5
  surveys a cluster of feed-forward 4D / temporal-Gaussian methods that the
  thread's current op set doesn't address: 4DGT, L4GM, 4D-LRM (offline
  feed-forward 4D) and StreamSplat / DGS-LRM (online streaming Gaussian
  motion). Question for the thread: does this warrant a new `op:dynamic-4d`
  operating point (subject to the ≤3 OPs cap — would likely displace
  `op:neural-free`, which is thinly supported)? Or is 4D feed-forward its
  own thread once enough mechanism-level papers land? Cross-reference with
  [[feed-forward-structure-from-motion]]'s temporal section.
- **Feed-forward Gaussian compaction as a candidate efficiency pathway.**
  [[wang2026_feed-forward-3d-scene-modeling]] §4.3.2 describes GGN,
  PixelGaussian, FreeSplat++, LongSplat as Gaussian-side compaction
  mechanisms (graph pooling, cascade pruning, triplet fusion, identity-aware
  redundancy compression) — complementary to the thread's current
  compression story (EA-3DGS codebook VQ). None yet in the wiki. Search
  target: the top 1–2 that report concrete city-scale / long-capture numbers.
- **Video-world vs 3D-world model paradigm** ([[wang2026_feed-forward-3d-scene-modeling]]
  §7.4): is the long-term role of radiance fields to serve as the persistent
  3D-world representation behind agents, or will video diffusion ("video as
  world simulator") absorb the function? This is a cross-thread framing
  shared with [[generative-3d-from-2d-priors]]. **Partial answer landed 2026-04-21**:
  [[shen2026_lyra2|Lyra 2.0]] demonstrates a production-grade video-world
  pipeline whose *output* is a radiance field (3DGS + mesh). So the paradigms
  are not zero-sum — the radiance field can sit *downstream* of a video-world
  generator as its persistent 3D state. Relevant to this thread at the
  `generative-reconstruction 3DGS output` boundary: [[downsampled-gaussian-dpt-head_shen2026]]
  is a candidate streaming-compatible primitive representation (see
  Candidate components) that is orthogonal to the thread's per-scene
  optimization OPs.
- **Feed-forward generative-reconstruction 3DGS as an alternative to per-scene
  optimization?** [[shen2026_lyra2|Lyra 2.0]] produces scene-scale 3DGS via
  a feed-forward lift on generated video rather than per-scene photometric
  optimization. None of the thread's current OPs (`op:quality-per-scene`,
  `op:city-scale`, `op:neural-free`) are feed-forward; all assume posed
  real images as input. Whether a future `op:generative-feed-forward` OP is
  warranted depends on additional video-world papers landing with directly
  comparable quality numbers — Lyra 2.0 alone is insufficient evidence for
  an OP split, and its input regime (single-image-driven generation) is
  sufficiently different from the current three OPs (posed-image
  reconstruction) that cross-OP benchmarking is hard.

## Related threads
- [[gaussian-to-mesh-pipelines]] — the downstream question of what to do with Gaussians
- [[feed-forward-structure-from-motion]] — feed-forward methods that may replace the SfM stage radiance fields depend on
- [[mono-depth-estimation]] — mono depth as a prior for sparse-view radiance field training

---

## SOTA pipelines

### op:quality-per-scene (object / room scale, PSNR-dominant)

3DGS primitives + MVS-depth geometry regularization + decoupled appearance:
- Primitive: 3DGS (Kerbl 2023).
- Geometry regularization: multiview-depth + median-depth relative loss. Paper: [[kim2025_multiview-geometric-gs]].
- Appearance decoupling: per-image CNN transforming target, not representation. Paper: [[lin2024_vastgaussian]] (generalized from NeRF-W).
- SfM/pose source: COLMAP (or InstantSfM when speed matters); CamP preconditioning when poses are noisy. Paper: [[park2023_camp]].

### op:city-scale (aerial / city; scaling-dominant)

3DGS + spatial partitioning + decoupled appearance + optional compression:
- Partition: airspace-aware `m×n` ground-plane cells with visibility-based camera selection. Paper: [[lin2024_vastgaussian]].
- Compression: codebook vector quantization + tetrahedral-mesh init. Paper: [[guo2025_ea-3dgs]]. Orthogonal to partitioning; stackable.
- Drone specifics: adaptive local-global distractor masking + MVS-voxel-guided sparse-view regularization. Paper: [[tang2025_dronesplat]].
- Incremental capture: voxel-based progressive 3DGS SLAM with loop closure. Paper: [[deng2026_vpgs-slam]].

### op:neural-free (sparse-voxel rasterization; natively mesh-extractable)

- Adaptive sparse voxels + Morton-ordered CUDA rasterizer + post-activation density. Paper: [[sun2025_sparse-voxels-rasterization]].
- Natively mesh-extractable (see [[gaussian-to-mesh-pipelines]] `op:natively-extractable`).

> **Note on the NeRF-family ceiling (Zip-NeRF stack).** The MipNeRF 360 + Zip-NeRF + CamP stack represents the implicit-NeRF quality ceiling — still load-bearing for reflective / unbounded benchmarks. After this migration it lives in `## Candidate components` rather than as its own OP, since the field has hand-ed off to 3DGS on edit-ability and real-time rendering. Re-promote to its own OP if a new implicit-NeRF paper moves the ceiling.

## Pipeline lineage

- 2020 · all OPs · baseline: volumetric rendering via MLP. Driver: NeRF.
- 2022 · (historical / NeRF-ceiling reference) · unbounded scenes: contract + proposal + distortion. Driver: [[barron2022_mip-nerf-360]].
- 2023 · (historical / NeRF-ceiling reference) · speed: hash-grid + anti-aliased cone. Driver: [[barron2023_zip-nerf]].
- 2023 · (historical / NeRF-ceiling reference) · implicit-NeRF ceiling: per-frame pose preconditioning. Driver: [[park2023_camp]].
- 2023 · op:quality-per-scene · representation switch: implicit MLP → explicit Gaussians. Driver: Kerbl (3DGS).
- 2024 · op:city-scale · new OP: partition + airspace-aware visibility + 2D-transform appearance. Driver: [[lin2024_vastgaussian]].
- 2025 · op:quality-per-scene · geometry: external MVS depth + median-depth relative loss. Driver: [[kim2025_multiview-geometric-gs]].
- 2025 · op:city-scale · compression: tetra-mesh init + codebook VQ. Driver: [[guo2025_ea-3dgs]].
- 2025 · op:city-scale · aerial robustness: adaptive masking + MVS-voxel regularization. Driver: [[tang2025_dronesplat]].
- 2025 · op:neural-free · new OP: sparse voxel rasterizer. Driver: [[sun2025_sparse-voxels-rasterization]].
- 2026 · op:city-scale · progressive voxel-GS with submaps + loop closure. Driver: [[deng2026_vpgs-slam]].

## Candidate components / not yet integrated

- **NeRF-family ceiling (Zip-NeRF stack)** — MipNeRF 360 contraction + Zip-NeRF hash-grid + CamP preconditioning. Parked here rather than as its own OP; still the quality ceiling on reflective / unbounded benchmarks. Papers: [[barron2022_mip-nerf-360]], [[barron2023_zip-nerf]], [[park2023_camp]].
- **Per-pixel confidence from CoMe** as an alternative to SSIM in VastGaussian's decoupled appearance module. Proposed in [[radl2026_confidence-mesh-3dgs]] discussion; no paper merges them. Target OP: `op:quality-per-scene`.
- **Pow3R-style prior injection** (intrinsics / poses / sparse depth) into a 3DGS training frontend. No paper does this yet; 3DGS is still COLMAP-posed. Target OP: all.
- **Feed-forward 3DGS initialization** (PixelSplat, MVSplat, NoPoSplat) replacing COLMAP-SfM in the per-scene pipeline. Target OP: `op:quality-per-scene`.
- **Foundation-feature distillation into per-Gaussian features** — tracked under [[lifting-foundation-models-to-3d]] but not yet in any Current-SOTA radiance pipeline.
- **[[downsampled-gaussian-dpt-head_shen2026]]** (Lyra 2.0) — `k × k` strided per-pixel Gaussian DPT head for 4× Gaussian reduction at prediction time. Target OP: `op:city-scale` (streaming-compatible Gaussian count is a live concern there) and `op:quality-per-scene` (real-time rendering at high resolution). **Did not displace any current filler** because (a) the current pipelines rely on per-scene optimization rather than a feed-forward Gaussian predictor, so the head-downsampling trick has no directly-comparable existing target; (b) no isolated ablation in the source paper isolates the head-modification gain from the surrounding fine-tune + video-model improvements. Revivable condition: a paper that drops the strided head into a pure feed-forward 3DGS baseline (MVSplat, pixelSplat) and reports per-pixel vs strided quality. This is orthogonal to EA-3DGS codebook VQ and to the Gaussian-side compaction family ([[wang2026_feed-forward-3d-scene-modeling]] §4.3.2) — prediction-time reduction + post-hoc compaction is a plausible stack.

## Open questions & synthesis bets

### Bet #001 — SVRaster + CoMe confidence + MVS-depth supervision
status: proposed
combines: [[sun2025_sparse-voxels-rasterization]], [[radl2026_confidence-mesh-3dgs]], [[schonberger2016_colmap-mvs]]
stage_target: radiance-fields.rendering
op_target: op:neural-free
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: SVRaster primitives paired with CoMe's per-primitive self-supervised confidence + external MVS depth could beat all 3DGS-based pipelines on geometry without losing render speed.
expected_gain: PSNR parity with op:quality-per-scene on NeRF-Synthetic / T&T + >0.03 F1 gain on geometry benchmarks, at SVRaster's render speed.
risk: CoMe's confidence is Gaussian-covariance-based; porting to voxel primitives requires a different uncertainty formulation. External MVS may conflict with voxel-native geometry.
validating_experiment: Drop-in CoMe-style confidence onto SVRaster; ablate with/without external MVS depth on T&T Intermediate. Compare vs. SVRaster baseline and 3DGS+CoMe baseline.
triggers: [ingest-of-idea:confidence-weighted-voxel-depth, benchmark:svraster-coome-ablation]
created: 2026-04-15 · updated: 2026-04-18

### Bet #002 — VastGaussian + EA-3DGS + VPGS-SLAM layered for city-scale
status: proposed
combines: [[lin2024_vastgaussian]], [[guo2025_ea-3dgs]], [[deng2026_vpgs-slam]]
stage_target: radiance-fields.partitioning
op_target: op:city-scale
confidence: med
magnitude: substantial
cost: months
breakage_risk: med
hypothesis: Partition at capture time (VPGS-SLAM submaps), compress per submap (EA-3DGS codebooks), merge at render time. No paper composes the three.
expected_gain: True km²-scale capture with render-time memory lower than any single-method approach; SSIM/LPIPS parity with VastGaussian on aerial benchmarks.
risk: Codebook quantization interacts badly with submap merging boundaries; may introduce visible seams. Loop-closure-driven submap edits invalidate codebooks.
validating_experiment: Sequential capture on UrbanScene3D (aerial); compare {Vast-only, Vast+EA, Vast+VPGS, Vast+EA+VPGS} on SSIM + memory + training time.
triggers: [ingest-of-idea:submap-boundary-codebook-alignment]
created: 2026-04-15 · updated: 2026-04-18

### Bet #004 — Calibrated mono-depth uncertainty as per-pixel weight for 3DGS depth supervision
status: proposed
combines: [[depth-proportional-uncertainty-fusion_pataki2025]], [[median-depth-relative-loss_kim2025]], [[va-gs-four-loss-stack_li2025]]
stage_target: radiance-fields.depth-supervision
op_target: op:quality-per-scene
confidence: med
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: 3DGS pipelines that supervise rendered depth against mono-depth priors (Kim 2025, VA-GS) currently treat the prior as a scalar loss term with per-image or median scaling. MP-SfM shows that off-the-shelf mono-depth uncertainties are badly calibrated and that a depth-proportional + model-uncertainty max recovers calibration. Plugging that recipe in as a per-pixel inverse-variance weight on the depth-supervision loss should reduce the loss's dominance in high-variance regions (distant objects, vegetation) where the mono prior is wrong anyway.
expected_gain: Less over-regularization on outdoor / vegetation scenes; +0.2–0.5 PSNR or improved geometry F-score on scenes where current median-relative loss visibly degrades the radiance field. Smaller gain expected on indoor scenes where mono-depth is already reliable.
risk: The calibration recipe was tuned for metric-depth SfM residuals, not photometric depth supervision. The optimal weight schedule may differ; uncertainty-aware loss can also vanish over training and stop contributing. Needs a modest calibration split; absent that, the depth-proportional term alone still helps.
validating_experiment: Replace fixed depth-supervision weight in Kim 2025 or VA-GS with per-pixel inverse variance from the MP-SfM recipe; re-run on Mip-NeRF 360 outdoor + ScanNet++ indoor; report PSNR, LPIPS, geometry F-score vs. baseline weighting.
triggers: [ingest-of-idea:depth-proportional-uncertainty-fusion_pataki2025]
created: 2026-04-21 · updated: 2026-04-21

### Bet #003 — CamP-style Jacobian preconditioner on 3DGS joint pose-refinement
status: proposed
combines: [[park2023_camp]], [[zhong2026_instantsfm]]
stage_target: radiance-fields.pose-joint-refine
op_target: op:quality-per-scene
confidence: high
magnitude: incremental
cost: days
breakage_risk: low
hypothesis: CamP's per-frame whitening of the camera Jacobian is representation-agnostic; applying it to 3DGS joint pose + Gaussians BA residual should give the same ~50% RMSE reduction CamP gave NeRF.
expected_gain: Reduced pose-refinement RMSE on noisy-pose datasets (e.g. Mip-NeRF 360 with synthetic pose jitter); +0.3–0.8 PSNR in the joint regime.
risk: 3DGS uses a different parameterization; whitening may conflict with the existing splatting gradient flow. CamP's Hessian-like conditioning cost may dominate the cheap 3DGS step.
validating_experiment: Implement whitening on InstantSfM's BA step with 3DGS downstream; ablate vs. un-preconditioned joint optimization. Report PSNR + pose-AUC.
triggers: [ingest-of-idea:3dgs-joint-ba-paper]
created: 2026-04-15 · updated: 2026-04-18

## Capability gaps

- **3DGS + joint BA pose refinement** — CamP's preconditioning trick has no 3DGS analog. Would unlock: robust per-scene 3DGS under noisy poses without leaning on COLMAP. Search target: 3DGS papers that jointly optimize camera + Gaussians with Jacobian preconditioning.
- **Automatic operating-point selector** — right now the user picks the OP (quality / city-scale / neural-free) by hand. Would unlock: one pipeline, adaptive partitioning. Search target: meta-papers that pick partitioning strategy from the input scene.
- **City-scale + compression + SLAM layered** — VastGaussian + EA-3DGS + VPGS-SLAM all exist, no paper composes the three. Would unlock: true km²-scale capture. Search target: any 2026 paper doing partition-while-capturing-while-compressing.

## Contradictions & tensions

- "External MVS depth > self-supervised 3DGS depth" (Kim 2025) vs. "self-supervised CoMe confidence wins" (Radl 2026) — same contradiction as in [[gaussian-to-mesh-pipelines]]. Open.
- Z-aliasing fix in Zip-NeRF is empirical with no principled explanation. No follow-up paper has closed this.

