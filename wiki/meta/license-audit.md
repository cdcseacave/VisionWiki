---
title: License Audit — commercial-use readiness of wiki papers
type: meta
tags: [license, audit, commercial-use, compliance]
created: 2026-04-18
updated: 2026-04-18
---

# License Audit

**Policy** (Hard Rule §6.15). **Licenses never block a bet or a SOTA-pipeline choice.** The priority is always the strongest pipeline mechanism-level reasoning can assemble. This audit exists to make **implementation-time** license planning explicit — nothing here downgrades any bet's `confidence`, changes any bet's `status`, or removes any idea from a thread's SOTA pipeline. Commercial deployment is a separate concern resolved at implementation time via one of: (a) separate-license negotiation with the authors / institution, (b) re-implementation of the mechanism under a permissive license (viable for the vast majority of ideas — they are mechanism-only, not data/weights-dependent), or (c) swap to a commercial-OK alternative.

**Scope.** All 55 wiki papers + 4 designs + 25 numbered bets (#001–#025 across all threads). Classifies each paper's code license into **commercial-OK** / **restricted** / **unknown** as a planning annotation. No "blocked" verdicts appear anywhere in this document — every bet stands regardless of license.

**Paper license note.** `arxiv-nonexclusive` + CC-BY / CC-BY-NC / CC-BY-SA on the *paper* constrains how the PDF + figures can be redistributed, not whether the ideas themselves can be reimplemented. Ideas are not patentable in most jurisdictions merely by publication. This audit therefore focuses on *code* licenses as the most informative planning signal.

---

## 1. Commercial-OK code (27 papers)

Papers whose authors' code is under a permissive license (MIT, Apache-2.0, BSD-3-Clause, GPL-3.0). These can be embedded in commercial pipelines with attribution only (or, for GPL-3.0, subject to copyleft).

| Paper | License | Notes |
|---|---|---|
| [bao2025_seg-wild](../papers/bao2025_seg-wild.md) | MIT | |
| [barron2022_mip-nerf-360](../papers/barron2022_mip-nerf-360.md) | Apache-2.0 | |
| [barron2023_zip-nerf](../papers/barron2023_zip-nerf.md) | Apache-2.0 | |
| [curless1996_tsdf](../papers/curless1996_tsdf.md) | MIT | reference impl (community) |
| [deng2026_vpgs-slam](../papers/deng2026_vpgs-slam.md) | MIT | |
| [edstedt2025_roma-v2](../papers/edstedt2025_roma-v2.md) | MIT | |
| [gao2025_anisdf](../papers/gao2025_anisdf.md) | MIT | |
| [jatavallabhula2023_conceptfusion](../papers/jatavallabhula2023_conceptfusion.md) | MIT | |
| [jin2026_zipmap](../papers/jin2026_zipmap.md) | Apache-2.0 | |
| [kirillov2023_sam](../papers/kirillov2023_sam.md) | Apache-2.0 | SAM 1 base, **not SAM 3** |
| [knapitsch2017_tanks-and-temples](../papers/knapitsch2017_tanks-and-temples.md) | MIT | dataset evaluation code |
| [li2025_megasam](../papers/li2025_megasam.md) | Apache-2.0 | |
| [lin2024_vastgaussian](../papers/lin2024_vastgaussian.md) | Apache-2.0 | authors released no code; community port is Apache-2.0 |
| [oquab2023_dinov2](../papers/oquab2023_dinov2.md) | Apache-2.0 | |
| [pan2024_glomap](../papers/pan2024_glomap.md) | BSD-3-Clause | |
| [park2023_camp](../papers/park2023_camp.md) | Apache-2.0 | |
| [pataki2025_mp-sfm](../papers/pataki2025_mp-sfm.md) | Apache-2.0 | |
| [radford2021_clip](../papers/radford2021_clip.md) | MIT | |
| [schonberger2016_colmap-mvs](../papers/schonberger2016_colmap-mvs.md) | BSD-3-Clause | COLMAP |
| [schonberger2016_colmap-sfm](../papers/schonberger2016_colmap-sfm.md) | BSD-3-Clause | COLMAP |
| [shi2024_open-vocab-segmentation](../papers/shi2024_open-vocab-segmentation.md) | Apache-2.0 | Trident |
| [tang2025_dronesplat](../papers/tang2025_dronesplat.md) | MIT | |
| [xie2025_gauss-mi](../papers/xie2025_gauss-mi.md) | GPL-3.0 | **copyleft — embedding creates derivative work obligations** |
| [yao2018_mvsnet](../papers/yao2018_mvsnet.md) | MIT | |
| [ye2024_gaussian-grouping](../papers/ye2024_gaussian-grouping.md) | Apache-2.0 | |
| [zhang2024_cameras-as-rays](../papers/zhang2024_cameras-as-rays.md) | MIT | |
| [zhao2025_diffusionsfm](../papers/zhao2025_diffusionsfm.md) | MIT | |

**GPL-3.0 caveat.** [xie2025_gauss-mi](../papers/xie2025_gauss-mi.md) imposes copyleft: any binary linking GPL-3.0 code and distributed outside the organization must ship source under GPL-3.0. Flagged as an implementation-time planning item — re-implement under a permissive license when embedding in a non-GPL product.

---

## 2. Restricted code (21 papers)

Non-commercial / research-only / custom-NOASSERTION code. **These do not block any bet or SOTA choice** — they are implementation-time annotations: at build time, pick one of {separate-license, re-implement, swap} per §6.15.

| Paper | License | Idea(s) used by which bet(s) / SOTA | Alternatives (if any) |
|---|---|---|---|
| [carion2026_sam-3](../papers/carion2026_sam-3.md) | SAM License (non-commercial) | Bets #013, #015, #017, #018, #020, #022; op:unified-promptable in `open-vocab-2d-composition` | SAM 1 (Apache-2.0) for basic segmentation — loses concept prompts + video tracks |
| [chen2024_pgsr](../papers/chen2024_pgsr.md) | ZJU research-only | Bet #005 | PGSR's mechanism (planar constraint + unbiased depth) is reimplementable under permissive license — no library dependency |
| [chen2025_sam-3d](../papers/chen2025_sam-3d.md) | SAM License (non-commercial) | Bets #022, #023; op:default in `generative-3d-from-2d-priors` | No commercial alternative for single-image generative 3D at this quality today |
| [chen2026_ttt3r](../papers/chen2026_ttt3r.md) | research-only | Bet #012; candidate in feed-forward-sfm | Mechanism is training-free & closed-form; reimplementable |
| [chebbi2025_multiview-dense-matching](../papers/chebbi2025_multiview-dense-matching.md) | DROID-SLAM-derived custom | Candidate in mvs feature-aggregation | MVSNet (MIT) or DUSt3R (CC-BY-NC on weights) as alternatives |
| [guedon2025_milo](../papers/guedon2025_milo.md) | Gaussian-Splatting-License (non-commercial) | Bet #004; op:mesh-in-loop in gaussian-to-mesh | MILo's mesh-in-loop idea is reimplementable but inherits 3DGS license constraints |
| [guo2025_ea-3dgs](../papers/guo2025_ea-3dgs.md) | Gaussian-Splatting-License | Bet #002; `op:city-scale` in radiance-field-evolution | Codebook VQ mechanism is reimplementable |
| [heinrich2025_radiov25](../papers/heinrich2025_radiov25.md) | NVIDIA Source Code License | Bets #014, #015, #016, #018, #020; `op:distilled-single-pass` | Student backbone weights are the value — re-distilling from scratch is feasible but expensive |
| [jang2025_pow3r](../papers/jang2025_pow3r.md) | CC-BY-NC-4.0 | Candidate in feed-forward-sfm | Pow3R's MLP-injection pattern is reimplementable |
| [li2025_geosvr](../papers/li2025_geosvr.md) | NVIDIA NSCL (inherits SVRaster) | Bet #006; `op:natively-extractable` in gaussian-to-mesh | Depends on SVRaster base; both are NVIDIA NSCL |
| [li2025_va-gs](../papers/li2025_va-gs.md) | **no LICENSE file** → all-rights-reserved | Bet #005 | Reimplementable; the four-loss stack is mechanism-only |
| [mao2025_spatiallm](../papers/mao2025_spatiallm.md) | Apache-2.0 code + CC-BY-NC-4.0 weights | Bets #024, #025 | Code is permissive; **weights are non-commercial** — re-train from the open synthetic dataset (also this paper) |
| [murai2025_mast3r-slam](../papers/murai2025_mast3r-slam.md) | custom (DROID-SLAM-derived) | Candidate in feed-forward-sfm | Verify specific DROID-SLAM terms before commercial use |
| [qin2024_langsplat](../papers/qin2024_langsplat.md) | Gaussian-Splatting-License | Bets #014, #020, #021; `op:per-scene-3dgs` in lifting-FM | LangSplat mechanism (per-scene autoencoder + SAM hierarchy) reimplementable, but 3DGS base license cascades |
| [radl2025_sof](../papers/radl2025_sof.md) | Gaussian-Splatting-License | Backing CoMe's meshing | Reimplementable under permissive license |
| [radl2026_confidence-mesh-3dgs](../papers/radl2026_confidence-mesh-3dgs.md) | CC-BY-NC-SA-4.0 (inherits 3DGS License) | Bets #001, #004, #005, #006, #014, #023; `op:regularized-3dgs` in gaussian-to-mesh | CoMe's self-supervised confidence is mechanism-only, reimplementable under permissive license |
| [simeoni2025_dinov3](../papers/simeoni2025_dinov3.md) | DINOv3 License (custom, commercial-permitted-with-distribution-terms) | Bets #007, #011, #015, #017, #018, #019, #020 | **Review distribution terms, often compatible with commercial use** — read the LICENSE carefully before assuming restricted |
| [sun2025_sparse-voxels-rasterization](../papers/sun2025_sparse-voxels-rasterization.md) | NVIDIA NSCL | Bets #001, #007; `op:neural-free` + `op:natively-extractable` | Mechanism reimplementable but rasterizer is custom CUDA — real work |
| [yu2025_cusfm](../papers/yu2025_cusfm.md) | NOASSERTION (custom, NVIDIA) | Bet #009; `op:sequential-slam-prior` in gpu-native-sfm | Review terms before commercial use |
| [zhang2025_loger](../papers/zhang2025_loger.md) | Apache-2.0 (code) + CC-BY-NC-4.0 (weights) | feed-forward-sfm candidate | Code permissive; weights non-commercial — retrain from scratch if needed |
| [zhong2026_instantsfm](../papers/zhong2026_instantsfm.md) | NOASSERTION (custom) | Bets #003, #008, #010; `op:general-purpose` in gpu-native-sfm | Review terms — paper says PyTorch-native, should be re-implementable |
| [zhu2025_gs-discretized-sdf](../papers/zhu2025_gs-discretized-sdf.md) | Gaussian-Splatting-License | `op:regularized-3dgs` candidate | Reimplementable |
| [wu2026_langsvr](../papers/wu2026_langsvr.md) | research-only (SVRaster inherited) | Bet #015; `op:voxel-online` | Depends on SVRaster base |

---

## 3. Unknown / ambiguous

None in the audited set — `unknown` has been pushed to 0 by `lint licenses` runs prior to this audit. Paper-side `license_paper: unknown` exists for 1 paper ([shi2025_clip-rectification] — if it exists; not in the current set) but paper licenses don't block commercial reimplementation of mechanisms.

---

## 4. Bet-level implementation-time license annotations

**Every bet below stands on its mechanism-level merit regardless of license** (§6.15). This table is a planning aid at implementation time — not a bet-readiness gate. "Implementation path" recommends which of {use-as-is, negotiate-license, re-implement, swap-component} is cheapest *at build time* when the bet moves from `feasible` → `in-design`.

| Bet | Papers in `combines:` | Restricted components | Implementation path (at build time, not bet-adoption time) |
|---|---|---|---|
| #001 | SVRaster + CoMe + COLMAP-MVS | SVRaster (NVIDIA NSCL), CoMe (3DGS-License) | Re-implement CoMe confidence atop a permissive SVRaster-style rasterizer — both mechanisms are well-specified. |
| #002 | VastGaussian + EA-3DGS + VPGS-SLAM | EA-3DGS (3DGS-License) | Re-implement codebook VQ under permissive license; VastGaussian and VPGS-SLAM are already Apache / MIT. |
| #003 | CamP + InstantSfM | InstantSfM (NOASSERTION) | Review InstantSfM LICENSE terms; if restrictive, the PyTorch-native design is straightforward to re-implement. |
| #004 | CoMe + MILo | CoMe + MILo (both 3DGS-License) | Both are mechanism-only; re-implement in a permissive 3DGS fork. |
| #005 | COLMAP-MVS + CoMe + PGSR + VA-GS | CoMe, PGSR, VA-GS | All three are mechanism-only; stack under permissive license. |
| #006 | GeoSVR + Kim 2025 + CoMe | GeoSVR (NSCL), CoMe | Mechanism re-implementation on permissive base. |
| #007 | SVRaster + DINOv3 | SVRaster, DINOv3 | DINOv3 License may permit commercial use — **read the LICENSE**; SVRaster mechanism is re-implementable. |
| #008 | InstantSfM + VastGaussian | InstantSfM | Review InstantSfM terms or re-implement its depth-constrained Jacobian. |
| #009 | CuSfM | CuSfM (NOASSERTION) | Review CuSfM LICENSE terms before choosing. |
| #010 | InstantSfM + MP-SfM + RoMa v2 | InstantSfM | Same as #003, #008. |
| #011 | DINOv3 + RoMa v2 | DINOv3 | Verify DINOv3 commercial-distribution terms. |
| #012 | TTT3R + RoMa v2 | TTT3R (research-only) | Training-free closed-form mechanism — trivially re-implementable. |
| #013 | Gaussian Grouping + SAM 3 | SAM 3 (SAM License) | Use SAM 3 under research license for R&D; for commercial, SAM 1 + a SigLIP concept head + a cross-view tracker is the fallback composition. |
| #014 | LangSplat + RADIOv2.5 + CoMe | all three | LangSplat autoencoder + CoMe confidence re-implementable; RADIOv2.5 student weights may need re-distillation or alternative backbone (DINOv3 + SigLIP) at commercial time. |
| #015 | LangSVR + SAM 3 + DINOv3 | LangSVR, SAM 3 | Re-implement LangSVR recipe; same SAM 3 fallback as #013. |
| #016 | ConceptFusion + RADIOv2.5 + VPGS-SLAM | RADIOv2.5 | Re-distill student from open teachers (CLIP + DINOv2 + SAM 1) if RADIOv2.5 weights are unusable commercially. |
| #017 | SAM 3 + DINOv3 + Trident | SAM 3 | Same SAM 3 fallback. |
| #018 | RADIOv2.5 + SAM 3 + DINOv3 | RADIOv2.5, SAM 3 | Re-distill + SAM 3 fallback. |
| #019 | SD-RPN + DINOv3 | DINOv3 | Verify DINOv3 terms. |
| #020 | SAM 3 + RADIOv2.5 + DINOv3 + LangSplat + Gaussian Grouping | SAM 3, RADIOv2.5, LangSplat | Multi-pronged re-implementation + SAM 3 fallback; acknowledged as a large build; confidence/magnitude/cost on the bet reflect build size, not license status. |
| #021 | GaussExplorer + LangSplat | LangSplat | Re-implement LangSplat autoencoder. |
| #022 | SAM 3D + SAM 3 | both | For R&D: use Meta weights; for commercial: re-derive single-image 3D flow-matching from open generative bases + SAM 1 masks. |
| #023 | SAM 3D + CoMe | SAM 3D, CoMe | Same as above for SAM 3D; CoMe mechanism-only. |
| #024 | SpatialLM | Apache-2.0 code + non-commercial weights | Commercial: re-fine-tune Qwen2.5 on the open synthetic indoor dataset (distributed with the paper). |
| #025 | SpatialLM + Gaussian Grouping | non-commercial SpatialLM weights | Same remediation; Gaussian Grouping is Apache-2.0 already. |

**Every bet is adopted on mechanism merit.** The restricted-components column is information only — it informs design-time staffing and timeline estimates (mechanism re-implementation adds engineering cost, flow into the bet's `cost:` field at design-time, but never into `status:` or `confidence:`).

---

## 5. Dataset licenses

Not separately audited — `license_dataset:` frontmatter field not consistently populated across dataset-referencing wiki pages. Known restrictive datasets in use:

- **Tanks and Temples**: MIT — commercial-OK (ground truth usable).
- **Mip-NeRF 360 scene dataset**: license not separately audited; standard academic benchmark terms apply.
- **ScanNet++**: custom research license; **verify commercial use before deployment**.
- **SA-Co / SA-1B** (SAM 3 / SAM training data): Meta non-commercial terms; inherited restriction when using the SAM-family weights.

Action: run a follow-up `lint licenses --include-datasets` to populate `license_dataset:` across all relevant papers.

---

## 6. Implementation-time checklist

These tasks are triggered when a bet reaches `feasible` → `in-design`. They are not prerequisites for bet adoption (§6.15).

- [ ] When a bet's build begins, review the restricted-components column above; pick cheapest path per component from {separate-license, re-implement, swap}.
- [ ] Read the DINOv3 License carefully — commercial use may already be permitted under the distribution terms. If confirmed, annotate affected ideas as `non-commercial-license-*` → `commercial-license-permitted-with-distribution-terms`.
- [ ] Review NVIDIA NSCL terms for SVRaster + RADIOv2.5 (may permit specific commercial use cases).
- [ ] Review InstantSfM / CuSfM NOASSERTION — these may be commercial-OK under authors' specific terms.
- [ ] Optionally contact VA-GS authors to request a LICENSE file (currently defaults to all-rights-reserved, which even blocks non-commercial re-use).
- [ ] Mechanism re-implementations under permissive licenses are cheap for: CoMe confidence, LangSplat autoencoder, MILo Delaunay-in-loop, PGSR planar + unbiased depth, EA-3DGS codebook VQ, CamP whitening, TTT3R closed-form LR, VA-GS four-loss stack, Kim 2025 median-depth relative loss. These can be scheduled into the `cost:` field of the bet's design rather than treated as license obstacles.
- [ ] Re-distillation paths (when original weights are restricted): RADIOv2.5 → re-distill from CLIP + DINOv2 + SAM 1 + SigLIP (all Apache-2.0 / MIT); SpatialLM → re-fine-tune Qwen2.5 on the open synthetic indoor dataset.
- [ ] Run `lint licenses --include-datasets` to close the dataset-side gap.
