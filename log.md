# Vision Wiki — Activity Log

Append-only, chronological, newest at bottom.
Entry prefixes are grep-friendly: `grep "^## \[" log.md | tail -20`.

## [2026-04-11] bootstrap
- Created: [CLAUDE.md](CLAUDE.md), [index.md](index.md), [log.md](log.md)
- Scaffolded: `raw/papers/`, `raw/assets/`, `wiki/{papers,methods,concepts,datasets,people,threads}/`
- Seeded: [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) as a placeholder for the NeRF → 3DGS lineage
- Notable: wiki begins empty of papers. Domain scope locked to photogrammetry + ML research.

## [2026-04-12] schema-change | Ingest now supports URLs and manages papers/
- Changed: [CLAUDE.md](CLAUDE.md) §1 (directory layout), §1.1 (new: paper storage), §3.1 (ingest workflow), §6.1 (hard rules)
- Added: top-level `papers/` directory with topic subfolders (radiance-fields, feature-matching, sfm-slam, mvs-depth, pose-estimation, mesh-reconstruction, fundamentals, datasets-benchmarks)
- Ingest now accepts: arXiv URLs, direct PDF URLs, project page URLs, or local file paths
- Papers auto-renamed to `<author>_<year>_<short-title>.<ext>` and filed into topic subfolders
- Wiki pages now link to local paper files via `local_paper:` frontmatter and inline links

## [2026-04-12] schema-change | Bare "ingest" = batch mode over raw/
- Changed: [CLAUDE.md](CLAUDE.md) §3.1 — added batch mode as default (no-argument) ingest behavior
- `raw/` now acts as an inbox: drop files there, say "ingest", all unprocessed papers get renamed/moved to `papers/<subfolder>/` and wiki-cascaded
- Originals deleted from `raw/` after confirmed placement in `papers/`
- Already-ingested files are skipped (detected via existing `wiki/papers/` page with matching `local_paper:`)

## [2026-04-12] schema-change | raw/ is pure inbox; added articles/ and assets/ top-level
- Changed: [CLAUDE.md](CLAUDE.md) §1 (layout), added §1.1 (inbox), §1.3 (articles), §1.4 (assets), §1.5 (classification rules)
- `raw/` is now exclusively an inbox — all file types accepted, all moved out on ingest
- `assets/` promoted to top-level (was `raw/assets/`) — permanent store for images/figures
- `articles/` added as top-level permanent store for blog posts, web articles, tutorials
- Classification rules: PDFs→papers/, markdown/HTML→articles/, images→assets/, ambiguous→ask user
- After complete ingest, `raw/` must be empty (this is the "unprocessed work" signal)

## [2026-04-12] seed | Three new research threads
- Created: [wiki/threads/gaussian-to-mesh-pipelines.md](wiki/threads/gaussian-to-mesh-pipelines.md), [wiki/threads/feed-forward-structure-from-motion.md](wiki/threads/feed-forward-structure-from-motion.md), [wiki/threads/mono-depth-estimation.md](wiki/threads/mono-depth-estimation.md)
- Updated: [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) (added cross-links to all three new threads)
- Updated: [index.md](index.md) (four threads now listed)
- Notable: these four threads form a connected graph — radiance fields depend on SfM for poses, mono depth for sparse-view priors, and need mesh extraction for downstream use

## [2026-04-12] ingest | Batch ingest of 32 papers from raw/
- **Filed**: 32 PDFs renamed and moved from `raw/` into `papers/{sfm-slam,mesh-reconstruction,radiance-fields,mvs-depth,pose-estimation,fundamentals,feature-matching}/`
- **Created (32 paper pages)**: wiki/papers/{zhong2026_instantsfm, pataki2025_mp-sfm, deng2026_vpgs-slam, yu2025_cusfm, murai2025_mast3r-slam, li2025_megasam, zhao2025_diffusionsfm, zhang2025_loger, jin2026_zipmap, li2025_geosvr, li2025_va-gs, gao2025_anisdf, radl2025_sof, guedon2025_milo, radl2026_confidence-mesh-3dgs, elflein2026_vgg-t3, mao2025_spatiallm, xie2025_gauss-mi, tang2025_dronesplat, park2023_camp, zhu2025_gs-discretized-sdf, sun2025_sparse-voxels-rasterization, kim2025_multiview-geometric-gs, guo2025_ea-3dgs, chebbi2025_multiview-dense-matching, jang2025_pow3r, zhang2024_cameras-as-rays, shi2026_self-distilled-roi, heinrich2025_radiov25, shi2024_open-vocab-segmentation, zhang2025_feed-forward-3d-survey, edstedt2025_roma-v2}.md
- **Created (method/concept stubs)**: ~17 stub pages in wiki/methods/ and wiki/concepts/ (3DGS, NeRF, COLMAP, DUSt3R, MASt3R, VGGT, DROID-SLAM, marching-cubes, 2DGS, bundle-adjustment, SDF, differentiable-rendering, test-time-training, monocular-depth-estimation, feature-matching, spherical-harmonics, vision-transformer)
- **Updated (4 threads)**: all threads populated with evidence from ingested papers
  - radiance-field-evolution: 8 papers, hypothesis updated, SVRaster flagged as dark horse
  - gaussian-to-mesh-pipelines: 10 papers, 4 competing paradigms identified (regularize/mesh-in-loop/native-mesh/feed-forward)
  - feed-forward-structure-from-motion: 12 papers, 3-tier landscape (accelerated classical/hybrid/fully feed-forward)
  - mono-depth-estimation: 6 papers, mono depth confirmed as load-bearing prior
- **Updated**: [index.md](index.md) fully rebuilt (32 papers, 4 threads)
- **raw/ status**: empty (all processed)
- Notable findings:
  - Test-time-training (LoGeR, ZipMap) emerging as the scaling trick for feed-forward reconstruction
  - Sparse voxel rasterization (SVRaster, GeoSVR) is a serious alternative to Gaussians for mesh-needed workflows
  - The DUSt3R/MASt3R family is the gravitational center of feed-forward SfM
  - MILo's mesh-in-the-loop is the most principled Gaussian-to-mesh approach but most complex

## [2026-04-12] schema-change | papers/ as git-ignored local cache + auto-download
- Changed: [CLAUDE.md](CLAUDE.md) §1.2 (papers/ is now a local cache, not committed), §3.1 Step 1 (cache-check + auto-download from url:), §3.2 (cache-check on query), directory layout diagram
- Created: [.gitignore](.gitignore) — excludes `papers/`, `assets/`, `.obsidian/workspace*`, `.DS_Store`
- `papers/` and `assets/` are now git-ignored (large binaries); wiki pages carry `url:` for re-download
- On fresh clone: papers are fetched on first access via `curl -L` from the arXiv URL in frontmatter

## [2026-04-12] enhancement | Added arXiv URLs to all 32 paper pages
- Updated: all 32 files in wiki/papers/ — added `url:` frontmatter field + `[arXiv]()` link
- Changed: [CLAUDE.md](CLAUDE.md) §2 (frontmatter template), §3.1 Step 3 (URL lookup requirement)
- URLs found via arXiv ID reconstruction (29 papers) + web search (3: MP-SfM, MegaSaM, LoGeR)
