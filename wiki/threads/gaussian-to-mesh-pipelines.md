---
title: Gaussian-to-Mesh Pipelines
type: thread
tags: [3dgs, mesh-reconstruction, surface-extraction, marching-cubes, sdf, tsdf, hierarchical-sparse-grid]
created: 2026-04-12
updated: 2026-04-21
sources: [papers/li2025_geosvr.md, papers/li2025_va-gs.md, papers/gao2025_anisdf.md, papers/radl2025_sof.md, papers/guedon2025_milo.md, papers/radl2026_confidence-mesh-3dgs.md, papers/elflein2026_vgg-t3.md, papers/kim2025_multiview-geometric-gs.md, papers/zhu2025_gs-discretized-sdf.md, papers/sun2025_sparse-voxels-rasterization.md, papers/chen2024_pgsr.md, papers/shen2026_lyra2.md]
operating_points: [op:regularized-3dgs, op:mesh-in-loop, op:natively-extractable]
status: draft
---

## Goal

From posed images + 3DGS primitives (or an alternative radiance representation), produce a watertight textured mesh. Better = higher T&T F-score (Intermediate + Advanced), lower DTU Chamfer, reasonable training time (<1h per scene for the lane that cares), no manual cleanup, rendering quality preserved if the method claims a joint appearance+geometry representation.

## Goal contract (optional, structured)

```yaml
metric: [T&T-F-score-intermediate, T&T-F-score-advanced, DTU-chamfer-mm, train-time-min, mesh-vertex-count]
target_regime: [posed-images, static-scene, bounded-or-unbounded, dense-to-sparse views]
constraints: [no-manual-cleanup, rendering-quality-preserved-if-joint, commercial-license-ok-preferred]
required_capabilities: [watertight-mesh-output, surface-normal-consistency, texture-preservation]
```

## Working hypothesis

The Gaussian-to-mesh problem has split into three competing paradigms:
**(A)** regularize 3DGS during training so Gaussians align with surfaces, then
extract via TSDF/marching-cubes; **(B)** co-optimize a mesh jointly with
Gaussians (mesh-in-the-loop); **(C)** bypass Gaussians entirely with
representations that are natively mesh-extractable (sparse voxels, SDF fields).
Paradigm A is the most popular (5+ papers), B is the most principled (MILo),
and C may win long-term (SVRaster, GeoSVR).

## Evidence

### Paradigm A: Regularize Gaussians → post-hoc mesh extraction

- [VA-GS (Li 2025)](../papers/li2025_va-gs.md): four alignment losses
  (edge-aware, visibility-aware, normal, feature) force Gaussians onto surfaces.
  Mesh via [[tsdf|TSDF]] fusion. Strong DTU results.
- [Kim 2025](../papers/kim2025_multiview-geometric-gs.md): MVS depth supervision
  + median-depth relative loss + normal/distortion regularization → SOTA explicit
  surface on DTU/TNT. Key insight: external MVS depth is more reliable than
  Gaussian self-supervision for geometry.
- [CoMe (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md): self-supervised
  per-Gaussian confidence scores balance photometric vs geometric loss +
  SSIM-decoupled appearance model. ~20 min runtime (vs MILo 60 min), SOTA on
  Tanks & Temples (F1 0.521) and ScanNet++.
- [Zhu 2025](../papers/zhu2025_gs-discretized-sdf.md): per-Gaussian discretized
  SDF values with learned SDF-to-opacity mapping. Enables both relighting and
  mesh extraction from the same representation.
- [PGSR (Chen 2024)](../papers/chen2024_pgsr.md): flattens Gaussians into
  planes + unbiased-depth rendering + multi-view geometric regularization.
  Matches neural-SDF surface quality without external priors — a strong
  argument that the 3DGS↔surface gap is a regularization problem, not a
  representation problem. Natural fit for Paradigm A.

### Paradigm B: Mesh-in-the-loop during training

- [MILo (Guedon 2025)](../papers/guedon2025_milo.md): Delaunay triangulation at
  every training step, differentiable mesh rendering in the loop. Result: 10x
  fewer mesh vertices than TSDF-based approaches, mesh quality comparable or
  better. Downside: more complex training pipeline.

### Paradigm C: Natively mesh-extractable representations

- [GeoSVR (Li 2025)](../papers/li2025_geosvr.md): explicit sparse voxels with
  uncertainty-weighted depth → direct [[marching-cubes]] extraction. No
  Gaussians at all. SOTA on DTU Chamfer distance.
- [SOF (Radl 2025)](../papers/radl2025_sof.md): sorted opacity fields with
  hierarchical resorting → 3x faster optimization, 10x faster meshing than
  standard 3DGS mesh pipelines. Robust marching-cubes from opacity fields.
- [SVRaster (Sun 2025)](../papers/sun2025_sparse-voxels-rasterization.md):
  sparse voxel rasterization is neural-free and produces meshes via standard
  marching-cubes. Matches 3DGS rendering quality.
- [AniSDF (Gao 2025)](../papers/gao2025_anisdf.md): fused-granularity SDF with
  anisotropic spherical Gaussians. Pure SDF approach — meshes come free via
  marching-cubes. Excels on reflective objects.

### Paradigm D: Feed-forward mesh prediction

- [VGG-T3 (Elflein 2026)](../papers/elflein2026_vgg-t3.md): replaces KV
  attention with test-time-trained MLPs for O(n) feed-forward 3D reconstruction.
  Outputs mesh/occupancy directly — no per-scene optimization needed.

## Emerging patterns

- **TSDF + marching-cubes** remains the dominant extraction path (used by VA-GS,
  Kim, SOF, and as a baseline in MILo).
- **Depth supervision from external MVS** consistently outperforms Gaussian
  self-supervision for geometry (Kim 2025, DroneSplat).
- **Confidence/uncertainty** is a recurring theme: Radl 2026 (per-Gaussian
  confidence), GeoSVR (voxel uncertainty), MILo (rendering confidence).

## Open questions
- Is there a representation that matches 3DGS rendering quality *and* gives
  watertight meshes without a separate extraction step? MILo comes closest
  but the training cost is higher.
- How do these methods compare under sparse-view regimes vs dense capture?
  Most benchmark on DTU (49 views) — few test at 3-10 views.
- What's the practical path from phone scan → splat → textured mesh for a
  game engine? Texturing is under-addressed — most papers stop at geometry.
- Will sparse voxels (GeoSVR, SVRaster) displace Gaussians for applications
  that need meshes?

## Related threads
- [[radiance-field-evolution]]
- [[feed-forward-structure-from-motion]] — VGG-T3 connects feed-forward 3D to mesh output
- [[mono-depth-estimation]] — mono depth priors improve Gaussian geometry
- [[nerfstudio]] — codebase map for the local 3DGS implementation

## Implementation notes
- [[come-integration-nerfstudio]] — design doc for porting CoMe into the local visiofacto fork

---

## SOTA pipelines

### op:regularized-3dgs (most popular; best when rendering quality matters)

Regularized 3DGS → TSDF/MC:
1. 3DGS training with geometric regularization (planar, normal, edge, visibility alignment losses). Paper: [[chen2024_pgsr]] (PGSR), [[li2025_va-gs]] (VA-GS).
2. External MVS depth supervision (via [[schonberger2016_colmap-mvs]] or learned MVS). Paper: [[kim2025_multiview-geometric-gs]].
3. Confidence-weighted depth rendering. Paper: [[radl2026_confidence-mesh-3dgs]] (CoMe) — current SOTA on T&T (F1 0.521).
4. TSDF fusion [[curless1996_tsdf]] + marching cubes.

### op:mesh-in-loop (best when mesh size matters)

1. Delaunay triangulation at every step. Paper: [[guedon2025_milo]] (MILo). 10× fewer vertices; comparable/better geometry; 3× longer training than CoMe.

### op:natively-extractable (best when Gaussians are optional)

1. Sparse voxels with uncertainty-weighted depth. Paper: [[li2025_geosvr]] (SOTA DTU Chamfer) or [[sun2025_sparse-voxels-rasterization]] (SVRaster).
2. Direct marching cubes on the voxel/SDF field. No Gaussians.

## Pipeline lineage

- 1996 · op:regularized-3dgs · surface fusion: ad-hoc range-image merging → TSDF weighted-average. Driver: [[curless1996_tsdf]].
- 2016 · op:regularized-3dgs · MVS depth source: baseline PatchMatch → COLMAP MVS with per-pixel view selection. Driver: [[schonberger2016_colmap-mvs]].
- 2018 · op:regularized-3dgs · MVS depth alternative: classical PatchMatch → learned plane-sweep. Driver: [[yao2018_mvsnet]].
- 2024 · op:regularized-3dgs · 3DGS regularization: unregularized splatting → planar-constrained + unbiased depth. Driver: [[chen2024_pgsr]].
- 2025 · op:regularized-3dgs · external vs. self-supervised depth: self-supervised wins visually, external MVS wins on geometry. Driver: [[kim2025_multiview-geometric-gs]] → formalized as "external MVS > Gaussian self-supervision for mesh quality".
- 2025 · op:mesh-in-loop · new OP: post-hoc TSDF → Delaunay-in-the-loop. Driver: [[guedon2025_milo]].
- 2025 · op:natively-extractable · new OP: Gaussians → sparse voxels with direct MC. Driver: [[li2025_geosvr]] / [[sun2025_sparse-voxels-rasterization]].
- 2026 · op:regularized-3dgs · confidence weighting: uniform fusion weight → per-Gaussian self-supervised confidence. Driver: [[radl2026_confidence-mesh-3dgs]].

## Candidate components / not yet integrated

- **TSDF fusion with per-Gaussian CoMe confidence as the weight** — the TSDF weighting slot exists since 1996; CoMe confidence exists since 2026; no paper combines them. Blocked on: implementation, not novelty.
- **MVSNet-family learned MVS as the depth source** for Pipeline A — every current Pipeline A paper uses classical COLMAP MVS or self-supervised depth. Blocked on: speed/quality comparison.
- **SAM 3 masks as geometry constraints** — on-surface vs. off-surface Gaussian supervision from segmentation. Blocked on: no paper.
- **[[hierarchical-sparse-grid-mesh-extraction_shen2026]]** (Lyra 2.0) — adaptive-level OpenVDB grid + per-level marching cubes + transition merging. OP considered: any — primarily relevant for *large scenes* (multi-room explorable walkthroughs, city-scale). **Did not displace any existing OP filler** because:
  - No isolated ablation in [[shen2026_lyra2|Lyra 2.0]] §4.4 — mesh extraction is described as implementation detail supporting Fig. 7's Isaac Sim demo, not benchmarked vs MILo/CoMe/TSDF baselines.
  - None of the thread's current benchmarks (DTU, T&T) target the scale where hierarchical allocation is load-bearing. Existing extraction fillers (TSDF, MILo-Delaunay, GeoSVR direct-MC) are evaluated at object / room scale where a single-level grid suffices.
  - Would win by default if the thread added a `op:large-scale` OP, since no current filler has demonstrated at multi-room / city-scene extent. See Capability gaps.
  - Revivable condition: a head-to-head benchmark vs MILo / CoMe at scene scale on a common ground-truth-mesh dataset.

## Open questions & synthesis bets

### Bet #004 — CoMe confidence as Delaunay vertex weighting in MILo
status: proposed
combines: [[radl2026_confidence-mesh-3dgs]], [[guedon2025_milo]]
stage_target: mesh-reconstruction.extraction
op_target: op:mesh-in-loop
confidence: high
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: MILo's Delaunay-in-the-loop training produces 10× fewer vertices than TSDF but trains 3× longer than CoMe. Using CoMe's per-Gaussian confidence as Delaunay vertex weighting could preserve MILo's vertex reduction with CoMe's speed.
expected_gain: MILo-quality geometry (T&T F1 parity) at CoMe-like training time (20 min vs. 60 min), 10× vertex reduction retained.
risk: Delaunay weighting interacts poorly with CoMe's photometric/geometric balance; may bias vertex density toward high-confidence regions at the cost of thin-structure coverage.
validating_experiment: Port CoMe's confidence module into MILo's Delaunay trigger; ablate weight scheduling on T&T Intermediate. Compare training time + vertex count vs. MILo baseline.
triggers: [ingest-of-idea:differentiable-weighted-delaunay]
created: 2026-04-15 · updated: 2026-04-18

### Bet #005 — Pipeline A with COLMAP-MVS depth + CoMe confidence + PGSR planar + VA-GS visibility
status: proposed
combines: [[schonberger2016_colmap-mvs]], [[radl2026_confidence-mesh-3dgs]], [[chen2024_pgsr]], [[li2025_va-gs]]
stage_target: radiance-fields.regularization
op_target: op:regularized-3dgs
confidence: med
magnitude: incremental
cost: weeks
breakage_risk: med
hypothesis: All four mechanisms exist but no paper stacks them. COLMAP-MVS as supervision, CoMe confidence as per-Gaussian weight, PGSR's planar constraint, and VA-GS's visibility loss each address a different failure mode (depth quality, confidence calibration, surface alignment, occlusion-induced floaters).
expected_gain: +0.02–0.05 F1 on T&T Advanced over CoMe baseline.
risk: Loss-term interference — four simultaneous constraints may over-regularize fine detail. Weight tuning is the real engineering cost.
validating_experiment: Systematic ablation {CoMe, CoMe+MVS, CoMe+PGSR, CoMe+VA-GS, full stack} on T&T Intermediate + Advanced.
triggers: [ingest-of-idea:multi-loss-3dgs-balance-scheduler]
created: 2026-04-15 · updated: 2026-04-18

### Bet #006 — GeoSVR + per-voxel CoMe-equivalent confidence + external MVS depth
status: proposed
combines: [[li2025_geosvr]], [[kim2025_multiview-geometric-gs]], [[radl2026_confidence-mesh-3dgs]]
stage_target: mesh-reconstruction.extraction
op_target: op:natively-extractable
confidence: med
magnitude: substantial
cost: weeks
breakage_risk: low
hypothesis: Put op:natively-extractable on top of op:regularized-3dgs's benchmarks by lifting CoMe-style confidence into voxels + external MVS depth supervision. GeoSVR already wins DTU Chamfer; adding confidence + external depth could also win T&T.
expected_gain: DTU Chamfer parity with GeoSVR + T&T F1 parity with CoMe, in a Gaussian-free pipeline.
risk: GeoSVR's own uncertainty mechanism may conflict with CoMe-style confidence. Depth supervision inside a voxel pipeline is untested.
validating_experiment: Implement per-voxel confidence field in GeoSVR; ablate with/without external MVS depth on DTU + T&T.
triggers: [ingest-of-idea:per-voxel-self-supervised-confidence]
created: 2026-04-15 · updated: 2026-04-18

### Bet #007 — Render-then-unwrap texturing with DINOv3 seam-minimization
status: proposed
combines: [[sun2025_sparse-voxels-rasterization]], [[simeoni2025_dinov3]]
stage_target: mesh-reconstruction.texturing
op_target: op:natively-extractable
confidence: med
magnitude: substantial
cost: months
breakage_risk: low
hypothesis: Every current Gaussian-to-mesh pipeline stops at geometry. Rendering views from the splat/voxel + UV-unwrapping with DINOv3 dense features as a seam-minimization signal could close the "deployment to game engine" gap.
expected_gain: Textured mesh output from 3DGS / SVRaster with visible seam count reduced vs. naive per-face projection; qualitative evaluation on phone-scan data.
risk: No standard quantitative metric for texture seam quality; publishable evaluation is hard. DINOv3 features are patch-level; may miss sub-patch seam artifacts.
validating_experiment: Render-then-unwrap a SVRaster mesh; compare {naive, MRF-smoothing, DINOv3-feature-cost} on visible seam count + perceptual study.
triggers: [ingest-of-idea:learned-texture-unwrapping]
created: 2026-04-15 · updated: 2026-04-18

## Capability gaps

- **Watertight mesh in one step without a separate extraction pass** — MILo is closest but 3× slower than TSDF-based. Would unlock: op:one-shot-mesh fourth OP or op:mesh-in-loop becoming the default. Search target: differentiable mesh-head architectures with sub-CoMe training cost.
- **Principled switch between external-MVS-depth and self-supervised-3DGS-depth** — external wins on controlled capture, self-supervised wins on sparse/textureless. No paper has a confidence-based switch. Search target: depth-reliability estimators that compose with both sources.
- **Texturing pipeline from splat/voxel to game-engine mesh** — every paper stops at geometry. Would unlock: deployment-ready pipelines. Search target: render-then-unwrap with foundation-feature seam-minimization.
- **Sparse-view benchmarks** (3–10 views) — most current benchmarks are dense-capture (DTU 49 views). Would unlock: real-world phone-scan regimes. Search target: sparse-view mesh benchmarks (ScanNet++, DL3DV subsets).
- **Large-scale scene extraction benchmark** — no in-wiki filler has been quantitatively demonstrated at scene scale (multi-room, multi-building). [[shen2026_lyra2|Lyra 2.0]]'s [[hierarchical-sparse-grid-mesh-extraction_shen2026]] is the first in-wiki filler targeting this regime but has no isolated ablation vs alternatives. Would unlock: a fourth OP (`op:large-scale-scene`) where hierarchical allocation is load-bearing. Search target: (a) a paper that explicitly benchmarks mesh-extraction methods at scene scale against a ground-truth mesh (Matterport3D-GT, Replica-GT, or a synthetic scene benchmark), or (b) a MILo / CoMe variant that scales to scene-level capture without memory blowup.

## Contradictions & tensions

- "External MVS depth > Gaussian self-supervision" (Kim 2025) conflicts with "Gaussian self-supervised confidence outperforms external supervision" (CoMe 2026). Resolution hypothesis: external depth wins when MVS quality is high (controlled capture); self-supervised wins when MVS is unreliable (sparse views, textureless surfaces). Open: nobody has a principled switch.

