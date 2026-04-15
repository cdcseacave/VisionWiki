---
title: Re-ingest tracker (deep-analysis + pipeline-synthesis workflow)
type: design
tags: [meta, tracker, ingest]
created: 2026-04-15
updated: 2026-04-15
sources: []
status: draft
---

# Re-ingest tracker

Systematic re-ingest of every paper in `papers/` under the enhanced ingest workflow (CLAUDE.md §3.1, Steps 2 + 5 Pass A/B). Triggered 2026-04-15.

## Rules of this batch

- **Step 2 hard gate**: post deep analysis to the user, wait for approval, then write.
- **Per-paper commit** on completion (after Step 8 report).
- **Retrofit threads as touched** to the new template (Goal & success criteria / Current SOTA pipeline / Pipeline lineage / Candidate components / Open questions & synthesis bets / Contradictions & tensions).
- **Log entries** include `Pipeline impact:` and `Synthesis bet:` lines.
- Between sessions: read this tracker, resume at first `☐` paper.

Status legend: `☐` not yet · `▶` in progress (analysis pending user review) · `✓` done.

## Batch 1 — Foundations (perception stack feeding everything downstream)

Primary threads: `foundation-features-for-geometry`, `open-vocab-2d-composition`

- ✓ radford_2021_clip
- ✓ oquab_2023_dinov2
- ✓ simeoni_2025_dinov3
- ✓ kirillov_2023_sam
- ✓ carion_2026_sam-3
- ✓ heinrich_2025_radiov25
- ✓ shi_2024_open-vocab-segmentation
- ✓ shi_2026_self-distilled-roi
- ✓ zhang_2025_feed-forward-3d-survey

## Batch 2 — Classical primitives & benchmarks (SOTA comparison baselines)

Primary threads: `gpu-native-sfm`, `gaussian-to-mesh-pipelines`, and all benchmark references.

- ✓ curless_1996_tsdf
- ✓ schonberger_2016_colmap-sfm
- ✓ schonberger_2016_colmap-mvs
- ✓ yao_2018_mvsnet
- ✓ knapitsch_2017_tanks-and-temples

## Batch 3 — Feature matching & dense matching

Primary thread: `foundation-features-for-geometry`

- ✓ edstedt_2025_roma-v2
- ✓ chebbi_2025_multiview-dense-matching

## Batch 4 — SfM / SLAM / feed-forward 3D

Primary threads: `feed-forward-structure-from-motion`, `gpu-native-sfm`, `mono-depth-estimation`

- ✓ pan_2024_glomap
- ✓ zhong_2026_instantsfm
- ✓ yu_2025_cusfm
- ✓ pataki_2025_mp-sfm
- ✓ zhang_2025_loger
- ✓ zhao_2025_diffusionsfm
- ✓ li_2025_megasam
- ✓ murai_2025_mast3r-slam
- ✓ chen_2026_ttt3r
- ✓ jin_2026_zipmap
- ✓ deng_2026_vpgs-slam
- ✓ jang_2025_pow3r
- ✓ elflein_2026_vgg-t3

## Batch 5 — Pose estimation

Primary thread: `foundation-features-for-geometry`

- ✓ zhang_2024_cameras-as-rays

## Batch 6 — Radiance fields core

Primary thread: `radiance-field-evolution`

- ✓ barron_2022_mip-nerf-360
- ✓ barron_2023_zip-nerf
- ✓ park_2023_camp
- ✓ lin_2024_vastgaussian
- ✓ sun_2025_sparse-voxels-rasterization
- ✓ guo_2025_ea-3dgs
- ✓ kim_2025_multiview-geometric-gs
- ✓ tang_2025_dronesplat

## Batch 7 — Gaussian → Mesh & SDF hybrids

Primary thread: `gaussian-to-mesh-pipelines`

- ✓ chen_2024_pgsr
- ✓ gao_2025_anisdf
- ✓ guedon_2025_milo
- ✓ li_2025_geosvr
- ✓ li_2025_va-gs
- ✓ radl_2025_sof
- ✓ radl_2026_confidence-mesh-3dgs
- ✓ zhu_2025_gs-discretized-sdf
- ✓ kim_2026_gauss-explorer
- ✓ xie_2025_gauss-mi

## Batch 8 — Semantic / open-vocab 3D

Primary threads: `lifting-foundation-models-to-3d`, `open-vocab-2d-composition`

- ✓ jatavallabhula_2023_conceptfusion
- ✓ qin_2024_langsplat
- ✓ ye_2024_gaussian-grouping
- ✓ bao_2025_seg-wild
- ✓ jiao_2025_clip-gs
- ✓ wu_2026_langsvr
- ✓ mao_2025_spatiallm
- ✓ chen_2025_sam-3d

## Progress log

(per-paper completion notes; updated at Step 8 of each ingest)

- **2026-04-15 — all 55 papers re-ingested.** Committed in 8 batches:
  - `c1c7b92` — Batch 1 (foundations, 9 papers)
  - `73bacb0` — Batch 2 (classical primitives + benchmarks, 5 papers)
  - `24b59a8` — Batch 3 (feature + dense matching, 2 papers)
  - `c884eff` — Batch 4 (SfM/SLAM/feed-forward 3D, 13 papers)
  - `e548e7a` — Batches 5+6 (pose estimation + radiance fields core, 9 papers)
  - `1efdb21` — Batch 7 (Gaussian→mesh & SDF hybrids, 10 papers)
  - final commit — Batch 8 (semantic / open-vocab 3D, 8 papers)
- Threads augmented with new SOTA-pipeline / lineage / candidate-components / synthesis-bets sections:
  foundation-features-for-geometry, open-vocab-2d-composition, gpu-native-sfm, gaussian-to-mesh-pipelines, radiance-field-evolution, lifting-foundation-models-to-3d (6 threads).
- Threads left in original form (already substantively covering the required content):
  feed-forward-structure-from-motion, mono-depth-estimation, nerfstudio.
