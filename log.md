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

## [2026-04-14] lint
- Scanned: 48 wiki pages
- Contradictions: 0 · Orphans: 31 · Missing pages (≥3 refs): 33 · Broken wikilinks: 392 · Frontmatter drift: 32 · Missing local papers: 0 · Broken md links: 0
- Root cause of most broken wikilinks + orphan inflation: case-sensitive Obsidian wikilinks (`[[COLMAP]]` vs. `colmap.md`).
- Followups suggested: apply alias rewrites, create stubs for GLOMAP / DINOv2 / structure-from-motion / multi-view-stereo / Zip-NeRF / TSDF / SAM / CLIP.

## [2026-04-14] lint apply
- Rewrote case-mismatched wikilinks to alias form (`[[colmap|COLMAP]]` etc.) across 22 files for: COLMAP, DUSt3R, MASt3R, NeRF, VGGT, DROID-SLAM, SDF, 2D/3D-Gaussian-Splatting.
- Created stubs:
  - wiki/methods/glomap.md
  - wiki/methods/dinov2.md
  - wiki/methods/zip-nerf.md
  - wiki/methods/sam.md
  - wiki/methods/clip.md
  - wiki/concepts/structure-from-motion.md
  - wiki/concepts/multi-view-stereo.md
  - wiki/concepts/tsdf.md
- Updated: index.md (added 8 entries, rebuild counts to 14 methods / 11 concepts).
- Notable: frontmatter drift (32 pages with empty `sources:` on paper pages) left intact — a paper is its own source, so empty arrays are semantically correct.

## [2026-04-14] ingest (batch, 7 papers for needs-source stubs)
- Downloaded: GLOMAP (2407.20219), DINOv2 (2304.07193), Zip-NeRF (2304.06706), SAM (2304.02643), CLIP (2103.00020), MVSNet (1804.02505), COLMAP-MVS (1607.08203) — all placed under the appropriate papers/ subfolders.
- Created paper pages: pan2024_glomap, oquab2023_dinov2, barron2023_zip-nerf, kirillov2023_sam, radford2021_clip, yao2018_mvsnet, schonberger2016_colmap-mvs.
- Expanded stubs: wiki/methods/{glomap,dinov2,zip-nerf,sam,clip}.md and wiki/concepts/{structure-from-motion,multi-view-stereo}.md — `sources:` populated, `status: draft`, `[!needs-source]` flag replaced by Key references section.
- Updated: wiki/methods/colmap.md (added Schönberger 2016 MVS + GLOMAP citations).
- Updated: index.md (+7 paper rows), paper count 33 → 40.
- Notable: arXiv 1607.08203 is Schönberger's **MVS** paper, not the SfM paper; the CVPR 2016 COLMAP SfM paper has no arXiv version. wiki/concepts/structure-from-motion.md retains a `[!needs-source]` flag for that.
- Notable: Curless & Levoy 1996 (TSDF) and KinectFusion have no arXiv — wiki/concepts/tsdf.md retains `[!needs-source]` with external references.
- raw/ confirmed empty after ingest.

## [2026-04-14] ingest (batch, 5 papers + 2 duplicates)
- raw/ scan: 7 files. Deleted duplicates (2103.00020v1 = CLIP, 2304.07193v2 = DINOv2; both already ingested).
- Ingested:
  - schoenberger2016sfm.pdf → papers/sfm-slam/schonberger_2016_colmap-sfm.pdf (**fills structure-from-motion needs-source**)
  - volrange.pdf → papers/mesh-reconstruction/curless_1996_tsdf.pdf (**fills tsdf needs-source**)
  - 2508.10104v1.pdf → papers/fundamentals/simeoni_2025_dinov3.pdf (DINOv3)
  - 2511.16719v2.pdf → papers/fundamentals/carion_2026_sam-3.pdf (SAM 3)
  - 586242263_*.pdf → papers/mesh-reconstruction/chen_2025_sam-3d.pdf (SAM 3D; no arXiv)
- Created paper pages: schonberger2016_colmap-sfm, curless1996_tsdf, simeoni2025_dinov3, carion2026_sam-3, chen2025_sam-3d.
- Updated: wiki/concepts/structure-from-motion.md (sources + needs-source removed), wiki/concepts/tsdf.md (sources + primary needs-source removed; KinectFusion still flagged), wiki/methods/colmap.md (SfM paper citation added), wiki/methods/dinov2.md (DINOv3 lineage), wiki/methods/sam.md (SAM 3 + SAM 3D lineage).
- Updated: index.md (+5 paper rows), paper count 40 → 45.
- Notable: SAM 3D has no arXiv preprint — distributed via Meta AI blog + code release. Used Meta project URL in frontmatter.
- Notable: The CVPR-only SfM paper gap in the wiki is now closed; all core classical pipeline references (SfM + MVS + TSDF + marching cubes origins) are present.
- raw/ confirmed empty.

## [2026-04-15] ingest (batch, 6 papers — 3DGS × foundation-model cluster)
- raw/ scan: 7 files (6 papers + 1 supplement).
- Ingested:
  - 2312.00732 → ye2024_gaussian-grouping (ECCV 2024; SAM-lifted per-Gaussian identities).
  - 2312.16084 → qin2024_langsplat (CVPR 2024; CLIP-distilled 3DGS language field).
  - 2406.06521 → chen2024_pgsr (TVCG 2024; planar 3DGS + unbiased depth → SOTA surfaces).
  - 2412.19142 → jiao2025_clip-gs (ICCV 2025; 3DGS-CLIP contrastive alignment for zero-shot 3D). + supplement.
  - 2507.07395 → bao2025_seg-wild (2025; interactive 3DGS segmentation for in-the-wild photos).
  - 2601.13132 → kim2026_gauss-explorer (2026; VLM + 3DGS for compositional embodied reasoning).
- Cascade: 3d-gaussian-splatting.md (+foundation-lifting section + PGSR section), gaussian-to-mesh-pipelines.md (+PGSR), sam.md (+3D lineage block), clip.md (+3D applications block).
- Updated: index.md (+6 rows), paper count 45 → 51.
- Notable: all 6 are relevant to the planned Phase-B "lifting foundation models to 3D" thread; Phase B will follow in this session.
- raw/ empty.

## [2026-04-15] schema-expand (Phase B — foundation-model + segmentation synthesis)
- Created 4 concept pages: foundation-model, self-supervised-learning, segmentation (taxonomy), open-vocabulary-segmentation.
- Created 3 thread pages: foundation-features-for-geometry, lifting-foundation-models-to-3d, open-vocab-2d-composition.
- Cascade: dinov2.md, clip.md, sam.md (link into concepts + threads), vision-transformer.md + feature-matching.md (link SSL + foundation-model + thread), paper-page backlinks on shi2024_open-vocab-segmentation, heinrich2025_radiov25, carion2026_sam-3, chen2025_sam-3d.
- Index: +4 concept rows, +3 thread rows; counts 14/11/6 → 14/15/9.
- Followup: KinectFusion still `[!needs-source]`. Consider future threads: foundation-model-distillation (RADIO family, needs ≥2 papers) and 3D-native multimodal pretraining (CLIP-GS + future).

## [2026-04-15] ingest (voxel lane of lifting-foundation-models-to-3d)
- Downloaded & ingested:
  - ConceptFusion (Jatavallabhula et al., RSS 2023, arXiv 2302.07241) → papers/radiance-fields/jatavallabhula_2023_conceptfusion.pdf
  - LangSVR (Wu et al., Huawei, arXiv 2602.15734, Feb 2026) → papers/radiance-fields/wu_2026_langsvr.pdf [dropped by user in raw/ during ingest]
- Created paper pages: jatavallabhula2023_conceptfusion, wu2026_langsvr.
- Cascade: lifting-foundation-models-to-3d.md gains a "Voxel-based lifting" subsection + two rows in the tradeoffs table; tsdf.md links ConceptFusion as a voxel-feature-fusion consumer.
- Updated: index.md (+2 rows), paper count 51 → 53.
- Notable: two voxel-representation data points (classical TSDF + sparse-voxel rasterization) now ground the voxel lane of the thread. The lifting pattern is confirmed representation-agnostic (Gaussian / sparse voxel / TSDF cell all admit the same "per-primitive foundation-feature" template).
- raw/ empty.
