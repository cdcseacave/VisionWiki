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

## [2026-04-12] query | Compare InstantSfM vs CuSfM + common principle
- Touched: [wiki/papers/zhong2026_instantsfm.md](wiki/papers/zhong2026_instantsfm.md), [wiki/papers/yu2025_cusfm.md](wiki/papers/yu2025_cusfm.md)
- Filed as: [wiki/threads/gpu-native-sfm.md](wiki/threads/gpu-native-sfm.md) — "Tier 1: Accelerated Classical SfM" thread
- Updated: [wiki/threads/feed-forward-structure-from-motion.md](wiki/threads/feed-forward-structure-from-motion.md) (cross-link to new thread), both paper pages (back-link to thread), [index.md](index.md)
- Notable: the two papers share a four-step playbook (keep classical math, move to GPU, add learned priors as constraints, 10x-40x speedup) while occupying opposite ends of the generalization-specialization axis

## [2026-04-13] schema-change | Added `designs/` wiki category and `design` page type
- Changed: [CLAUDE.md](CLAUDE.md) §1 (layout — added `wiki/designs/`), §2 (frontmatter `type:` now includes `design`)
- Rationale: CoMe-into-nerfstudio implementation blueprint didn't fit under papers/methods/concepts/threads — it's a "how to build it" document. Design docs will accumulate as implementation plans are drafted.

## [2026-04-13] research-and-design | CoMe confidence + variance losses → nerfstudio/visiofacto
- Added alias `CoMe` + `project_page: https://r4dl.github.io/CoMe/` to [wiki/papers/radl2026_confidence-mesh-3dgs.md](wiki/papers/radl2026_confidence-mesh-3dgs.md)
- Created: [wiki/threads/nerfstudio.md](wiki/threads/nerfstudio.md) — codebase thread for `/Users/dancostin/Pro/nerfstudio` (visiofacto fork) + `/Users/dancostin/Pro/gsplat` submodule. Documents Pipeline→Model→Trainer architecture, `gauss_params` per-Gaussian state model, loss dispatch in `get_loss_dict()`, gsplat's 3-stage rendering pipeline, and the "feature channel trick" for adding arbitrary per-Gaussian renderable scalars.
- Created: [wiki/designs/come-integration-nerfstudio.md](wiki/designs/come-integration-nerfstudio.md) — 6-phase implementation blueprint covering config additions, per-Gaussian `γ_i` plumbing, confidence-map rendering via feature-channel trick, `L_conf` loss, confidence-aware densification threshold, normal variance loss (closed-form `1−||N||²`), color variance loss (second-moment trick). Identifies color variance as the only contribution that may need kernel work, with a pure-Python alternative via `Σw_i||c_i||²`.
- Cross-linked from the CoMe paper page + [gaussian-to-mesh thread](wiki/threads/gaussian-to-mesh-pipelines.md)
- Notable finding: gsplat's `rasterization()` accepts arbitrary feature channels (up to 32 native, unlimited via `channel_chunk`) — this means ~90% of CoMe can be implemented in pure Python without touching gsplat CUDA kernels.

## [2026-04-14] query | Does CoMe require per-pixel SSIM map?
- Touched: [wiki/papers/radl2026_confidence-mesh-3dgs.md](wiki/papers/radl2026_confidence-mesh-3dgs.md), raw paper text
- Updated: [wiki/designs/come-integration-nerfstudio.md](wiki/designs/come-integration-nerfstudio.md) — Phase 3 section expanded with explicit SSIM-map warning, concrete `ssim_per_pixel` code, and new validation check ("SSIM map equivalence at Ĉ=1")
- Answer: yes. `Ĉ ∈ ℝ^{H×W}` forces `L_rgb` to also be a 2D map; the standard scalar SSIM API silently broadcasts and computes the wrong thing. Implementation requires exposing SSIM's pre-reduction map — a one-time refactor to `model_components/losses.py`.
- Notable: this is the kind of detail the paper states implicitly (via Ĉ's shape) but never warns about; catching it in the design doc prevents a silent-wrong-loss bug that would only manifest as degraded F1 numbers with no error.

## [2026-04-14] design-update | Phase 3 now fully specifies per-pixel SSIM integration
- Updated: [wiki/designs/come-integration-nerfstudio.md](wiki/designs/come-integration-nerfstudio.md) Phase 3 rewritten to lead with the full per-pixel loss formulation (Eq. 4 + Eq. 9 derivation, complete math for L_1 map, D-SSIM map, L_rgb map, L_conf scalar), followed by the ssim_map helper, the visiofacto wire-up, and the one equivalence invariant (ssim_map(...).mean() == baseline scalar SSIM).
- Added: optional SSIM-decoupled variant for when §3.2 of the paper is also implemented — L_D-SSIM^dec uses l/c/s component maps from the same sliding window.
- Simplified: validation plan (removed duplicate SSIM-check bullet that 2.4.4 already covers).

## [2026-04-14] ingest | VastGaussian (Lin 2024, CVPR 2024)
- Acquired: [papers/radiance-fields/lin_2024_vastgaussian.pdf](papers/radiance-fields/lin_2024_vastgaussian.pdf) (moved from raw/, arXiv 2402.17427)
- Created: [wiki/papers/lin2024_vastgaussian.md](wiki/papers/lin2024_vastgaussian.md)
- Updated: [wiki/methods/3d-gaussian-splatting.md](wiki/methods/3d-gaussian-splatting.md) (new "Scaling to large scenes" section covering VastGaussian / EA-3DGS / VPGS-SLAM as three orthogonal attacks on large-scene 3DGS)
- Updated: [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) (expanded "Scaling: outdoor and large scenes" section; partially struck the "can 3DGS scale without quantization?" open question; added new open question on portability of decoupled appearance modeling to other rasterizers)
- Updated: [index.md](index.md) (new paper entry, 3DGS method promoted from stub, thread date bumped, 32→33 papers)
- Notable: airspace-aware visibility is the subtle key insight — defining cell visibility over the vertical column (not surface convex hull) is what prevents floaters, because floaters live *above* the surface. Decoupled appearance modeling (CNN + per-image embedding → transformation map applied only to training targets, discarded at inference) is the rasterization-compatible successor to NeRF-W's GLO embeddings.

## [2026-04-14] ingest-followup | VastGaussian ↔ CoMe cross-link
- Updated: [wiki/papers/radl2026_confidence-mesh-3dgs.md](wiki/papers/radl2026_confidence-mesh-3dgs.md) — SSIM-decoupled appearance model section now explicitly frames CoMe as a refinement of VastGaussian's L1/D-SSIM split; added VastGaussian to sources[].
- Updated: [wiki/papers/lin2024_vastgaussian.md](wiki/papers/lin2024_vastgaussian.md) — limitations section notes that CoMe later addressed the "D-SSIM can still absorb contrast/structure drift" weakness by decoupling SSIM into l·c·s.
- Notable: this is exactly the lineage CLAUDE.md §3.4 wants captured — CoMe (2026) is a *superseding refinement* of VastGaussian's (2024) appearance module, not an independent idea. The link now runs both directions.

## [2026-04-14] design-update | Add Part 7 — appearance module integration
- Updated: [wiki/designs/come-integration-nerfstudio.md](wiki/designs/come-integration-nerfstudio.md) — new **Part 7** (Phases 7A–7F) specifies porting VastGaussian's decoupled appearance module (per-image embedding + CNN transformation map) into visiofacto, with CoMe's SSIM-decoupled refinement so the embedding corrects luminance only while contrast/structure keep supervising geometry. Full CNN spec (VastGaussian Fig. 9), exact loss recipe composing with Phase 3's confidence-weighted per-pixel `L_rgb`, `ssim_components(a,b)→(l,c,s)` helper with `l*c*s == ssim_map` invariant, and wiring that keeps multiview photometric losses on untransformed renders.
- Updated scope disclaimer (top of doc): removed "out of scope" note on appearance/SSIM decoupling, renumbered goals 1–3, sources[] now includes VastGaussian.
- Updated: [wiki/papers/lin2024_vastgaussian.md](wiki/papers/lin2024_vastgaussian.md) — new *Downstream in this wiki* section points to the design doc.
- Updated: [index.md](index.md) — design entry scope expanded, date bumped.
- Notable: the confidence × appearance composition is clean because both act on a per-pixel `L_rgb` map — answered Part 5's open question #1 about how multiview + confidence nest. Open design decision: eval with `M=1` (cheap, geometry-focused) vs paper-faithful half-image-fit protocol (expensive, requires mid-eval optimization); recommended deferring the latter until we need to publish numbers against VastGaussian.

## [2026-04-14] design-correction | Part 7 — practical details from CoMe §A.1 / §A.2 appendix
- Updated: [wiki/designs/come-integration-nerfstudio.md](wiki/designs/come-integration-nerfstudio.md)
- Re-read CoMe §3.2, §A.1, §A.2, §A.3, §B.3, §C.1 and discovered five material corrections missing from the first draft:
  1. Transform uses **`exp(M)`**, not sigmoid — sigmoid cannot compensate over-exposure. Paired with zero-init (W=b=0) on the final conv so exp(M)=1 at t=0 (identity).
  2. CNN input is **detached** (`ds_32(Î).detach()`) — gradients of the appearance embedding must not flow back into the 3DGS render.
  3. CoMe adds a **(u, v, r) positional encoding** to the CNN input → 70 channels instead of 67; critical for learning vignetting.
  4. **Reflection padding** to next multiple of 32, crop back after — replaces VastGaussian's center-crop.
  5. **Custom fused CUDA kernel** gives 5× speedup (out of scope for first integration).
- Also corrected Phase 3: confidence clamp is **[1e-3, 5.0]** both ends (upper clamp prevents over-densification of already-good regions, paper is explicit).
- Also corrected Phase 4: densification divisor is **`min(γ̃, 1)`** clamped from above (my first draft had clamp from below only — confident Gaussians would have incorrectly gotten an *easier* split threshold).
- Added new §7.1.3 "Why the CNN operates on a 32× downsampled image (not full-res)" — two cooperating justifications (efficiency + semantic frequency bound). The semantic reason (downsample prevents M from absorbing high-frequency geometric detail) is the load-bearing design insight, not just efficiency.
- Added Phase 6 note: Option A's closed-form (`Σw_i||c_i||² − ||C||²`) is mathematically equivalent to CoMe Eq. 13 — both compute the same loss, CoMe just uses a fused CUDA kernel for speed.
- Notable: CoMe §A.1 footnote states "the source code for VastGaussian [35] was never released" — confirmed that CoMe's appendix is the authoritative reference implementation for the appearance module, not the VastGaussian paper alone.
