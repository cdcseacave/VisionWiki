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

## [2026-04-15] lint
- Orphans: 22 · Broken wikilinks: 189 · Missing pages ≥3 refs: 15 · Missing local papers: 0
- Frontmatter: 52 paper pages with empty `sources:` (schema ambiguity, not drift)

## [2026-04-15] schema-change
- Clarified `sources:` semantics in CLAUDE.md §2: paper pages may have empty `sources:` (a paper page is its own source). Required non-empty for method/concept/thread/dataset/person/design pages only.

## [2026-04-15] lint stubs
- Created 15 stubs for ≥3-ref missing pages:
  - methods: pow3r, neus, mip-nerf-360, metric3dv2, scaffold-gs, sgm, cut3r, croco, raydiffusion, lightglue, dpt, vggsfm, depthanything
  - concepts: neural-implicit-surfaces, levenberg-marquardt
- Each stub cites the referring paper(s) in `sources:`; primary-reference gaps flagged with [!needs-source]
- Resolved 13 of the top missing-page broken wikilinks

## [2026-04-15] lint orphans
- Wired all 22 orphan paper pages into existing threads via "## Related papers — pending integration" sections
- Threads touched: radiance-field-evolution, feed-forward-structure-from-motion, gpu-native-sfm, gaussian-to-mesh-pipelines, mono-depth-estimation, foundation-features-for-geometry, open-vocab-2d-composition
- Orphans remaining: 0

## [2026-04-15] lint orphans — deep integration
- Replaced the shallow "## Related papers — pending integration" queues with proper narrative integration across 7 threads.
- Finding: 19 of 22 papers were already integrated in thread bodies via relative markdown links; the previous orphan detector only counted [[wikilinks]], so those papers looked orphaned when they weren't. Fixed the detector (now counts markdown-style paper links too): true orphan count is 0.
- radiance-field-evolution: added Zip-NeRF as the pre-3DGS anchor in the NeRF→3DGS transition section; CamP commentary extended. CamP/EA-3DGS/DroneSplat/GauSS-MI/VPGS-SLAM already integrated.
- feed-forward-structure-from-motion: no body change needed — all 9 papers already in narrative. Queue deleted.
- gpu-native-sfm: added a "Classical baseline being accelerated" section with strengths/weaknesses of both Schönberger 2016 COLMAP papers, framing the Tier-1 motivation.
- gaussian-to-mesh-pipelines, mono-depth-estimation, foundation-features-for-geometry: bodies already integrated; queues deleted.
- open-vocab-2d-composition: added an "Adjacent patterns" section drawing the connection between SD-RPN's self-distilled MLLM attention and the CLIP+DINO+SAM composition pattern — honest placement rather than forcing.
- Schema observation: lint orphan-detector should count both [[wikilinks]] and relative markdown links; current schema uses markdown links for paper citations (§2).

## [2026-04-15] schema-change
- CLAUDE.md §3.3: clarified that `lint` orphan detection must count BOTH wikilinks (`[[slug]]`) and relative markdown wiki-page links (`[text](../papers/slug.md)`). Earlier detector only counted wikilinks, falsely flagging 19 correctly-cited paper pages as orphans.

## [2026-04-15] ingest | TTT3R, Mip-NeRF 360, Tanks and Temples
- Batch ingest from raw/ (3 files).
- Created: wiki/papers/chen2026_ttt3r.md, wiki/papers/barron2022_mip-nerf-360.md, wiki/papers/knapitsch2017_tanks-and-temples.md, wiki/datasets/tanks-and-temples.md (first entry in wiki/datasets/)
- Promoted: wiki/methods/mip-nerf-360.md from stub → stable (full method narrative)
- Updated: wiki/threads/feed-forward-structure-from-motion.md (TTT3R added to Tier-3 alongside LoGeR/ZipMap; emerging-patterns updated to frame TTT3R as the training-free point in the TTT design space), wiki/threads/foundation-features-for-geometry.md (TTT3R cited as evidence that frozen-backbone composability extends to training dynamics), wiki/threads/radiance-field-evolution.md (Mip-NeRF 360 added as the last implicit-NeRF frontier mover, anchoring the NeRF→3DGS transition), wiki/papers/barron2023_zip-nerf.md (sources: populated with Mip-NeRF 360 predecessor; citation link added), index.md (new paper entries + first Datasets entry).
- Files moved: raw/2509.26645v4.pdf → papers/sfm-slam/chen_2026_ttt3r.pdf; raw/2111.12077v3.pdf → papers/radiance-fields/barron_2022_mip-nerf-360.pdf; raw/tanks-and-temples.pdf → papers/datasets-benchmarks/knapitsch_2017_tanks-and-temples.pdf.
- Notable: TTT3R is the cleanest training-free point in the TTT-for-long-context design space (LoGeR/ZipMap required new training); its 2× CUT3R pose improvement is a free lunch that any deployed CUT3R should apply. Mip-NeRF 360 paper's distortion regularizer is a rare first-principles ambiguity fix — worth a revisit when designing regularizers for 3DGS/sparse-voxel mesh extraction. T&T is the 9-year-old benchmark still unsaturated on its Advanced split.
- raw/ empty after ingest.

## [2026-04-15] schema-change
- CLAUDE.md §3.3: added `lint stale-threads` sub-action. Definition: a thread is stale when `thread.updated < max(source.updated)` for sources listed in its frontmatter. Diagnostic in bare `lint`; targeted re-cascade procedure (with per-thread approval) as `lint stale-threads`. Replaces the vaguer "thread debt: not updated in last N ingests" bullet with a deterministic, computable check.

## [2026-04-15] schema-change | Ingest elevated to deep analysis + pipeline synthesis
- CLAUDE.md §2, §3.1 Step 2, §3.1 Step 5, §3.1 budget, §5: ingest reframed around deep mechanism-level analysis (Step 2 hard gate) and two-pass thread evolution (Pass A per-stage, Pass B holistic synthesis). Added `## Pipeline contribution` to paper template; rewrote Thread template as a living SOTA pipeline (Goal / Current SOTA pipeline / Pipeline lineage / Candidate components / Open questions & synthesis bets / Contradictions). Log entries now carry `Pipeline impact` and `Synthesis bet` lines.
- README.md: reframed project as a "research-synthesis engine" whose end-product is novel pipelines, not summaries.
- CLAUDE.md §1: project description updated to reflect the new goals.

## [2026-04-15] reingest-batch | Batch 1 — Foundations (9 papers)
- Created: wiki/designs/reingest-tracker.md
- Re-ingested under enhanced workflow: radford2021_clip, oquab2023_dinov2, simeoni2025_dinov3, kirillov2023_sam, carion2026_sam-3, heinrich2025_radiov25, shi2024_open-vocab-segmentation, shi2026_self-distilled-roi, zhang2025_feed-forward-3d-survey
- Added `## Pipeline contribution` sections to all 9 papers enumerating per-contribution mechanism, target thread, stage, and expected gain.
- Threads augmented with new structural sections:
  - foundation-features-for-geometry: added Goal & success criteria, Current SOTA pipeline (DINOv3 backbone + task head), Pipeline lineage (SIFT → DINOv2 → DINOv3 → RADIOv2.5 for multi-task; TTT3R as training-dynamics source), Candidate components (SigLIP, EVA-CLIP, 3D-native SSL), Open questions & synthesis bets (silent-failure rejection head, TTT3R + RoMa v2 fusion).
  - open-vocab-2d-composition: added three parallel Current SOTA pipelines (Trident / SAM 3 / RADIOv2.5), Pipeline lineage, Candidate components (SigLIP, SAM 2), three explicit synthesis bets (SigLIP+DINOv3+SAM 3 Trident, SAM 3+DINOv3 RADIO distillation, SD-RPN spatial denoising on DINOv3), Contradictions (benchmark incommensurability).
- Pipeline impact: foundation-features-for-geometry: backbone stage formalized (DINOv3 default) + 1 synthesis bet · open-vocab-2d-composition: 3 pipelines formalized + 3 synthesis bets · Pass B on all 9 papers.
- Synthesis bets proposed: DINOv3 self-attention → SD-RPN-style denoised spatial signal · TTT3R closed-form LR applied to RoMa v2 dense matching · RADIOv2.5-into-LangSplat per-Gaussian distillation · SAM 3 instance IDs replace Gaussian Grouping cross-view association.
- Notable: CLIP explicitly excluded from foundation-features-for-geometry (text-aligned/spatially weak); SigLIP flagged as highest-priority missing stub.

## [2026-04-15] reingest-batch | Batch 2 — Classical primitives & benchmarks (5 papers)
- Re-ingested: curless1996_tsdf, schonberger2016_colmap-sfm, schonberger2016_colmap-mvs, yao2018_mvsnet, knapitsch2017_tanks-and-temples
- Added `## Pipeline contribution` to each; for the benchmark paper (T&T) the section describes its role in defining success criteria, not components.
- Threads augmented with new sections:
  - gpu-native-sfm: Goal & success criteria, Current SOTA pipeline, Pipeline lineage (2016 COLMAP → 2024 GLOMAP → 2026 InstantSfM / CuSfM), Candidate components (learned feature frontends, Pow3R conditioning), synthesis bets (differentiable SfM+radiance-field backprop; CuSfM two-stage factor graph on non-driving; multi-prior Jacobian fusion), Contradictions.
  - gaussian-to-mesh-pipelines: three Current SOTA pipelines (regularized-3DGS+TSDF / mesh-in-loop / natively-extractable), lineage, three synthesis bets (CoMe confidence as TSDF fusion weight; PGSR+VA-GS+CoMe+COLMAP-MVS stack; GeoSVR+confidence+MVS supervision), Contradictions (external-vs-self-supervised depth).
- Pipeline impact: gpu-native-sfm: baseline formalized + 3 bets · gaussian-to-mesh-pipelines: 3 pipelines formalized + 3 bets · Pass B on all 5 papers.
- Synthesis bets proposed: CoMe-confidence as TSDF fusion weight · PGSR+VA-GS+CoMe stacked regularization · MILo + CoMe hybrid · differentiable joint SfM+3DGS training · MVSNet learned-MVS inside CoMe pipeline.
- Notable: explicit "external vs. self-supervised depth" contradiction flagged in gaussian-to-mesh-pipelines — unresolved; candidate for a future thread-level resolution.

## [2026-04-15] reingest-batch | Batch 3 — Feature matching & dense matching (2 papers)
- Re-ingested: edstedt2025_roma-v2, chebbi2025_multiview-dense-matching
- Added `## Pipeline contribution` to both.
- Pipeline impact: foundation-features-for-geometry task-head slot formalized around RoMa v2; covariance output identified as the missing input for [[gpu-native-sfm]]'s multi-prior-Jacobian synthesis bet. Chebbi adds an aerial/satellite MVS lane as a candidate depth source for VastGaussian/DroneSplat preprocessing.
- Synthesis bet proposed: RoMa v2 covariance → InstantSfM Jacobian weighting (mixes [edstedt2025_roma-v2] + [zhong2026_instantsfm]).

## [2026-04-15] reingest-batch | Batch 4 — SfM / SLAM / feed-forward 3D (13 papers)
- Re-ingested: pan2024_glomap, zhong2026_instantsfm, yu2025_cusfm, pataki2025_mp-sfm, zhang2025_loger, zhao2025_diffusionsfm, li2025_megasam, murai2025_mast3r-slam, chen2026_ttt3r, jin2026_zipmap, deng2026_vpgs-slam, jang2025_pow3r, elflein2026_vgg-t3
- Added `## Pipeline contribution` sections to all 13; threads [[feed-forward-structure-from-motion]] and [[gpu-native-sfm]] already carry the Current-SOTA-pipeline / lineage / synthesis-bet sections from Batches 1–3.
- Pipeline impact: Tier-1 SOTA pipeline (GPU-native SfM) formalized with GLOMAP→InstantSfM/CuSfM; Tier-3 TTT-family formalized (TTT3R/LoGeR/ZipMap trade triangle: training-free / training-with-SWA / training-with-TTT-attention-replacement); Tier-2 hybrid formalized with MP-SfM / MegaSaM / Pow3R.
- Synthesis bets proposed: LoGeR SWA + TTT3R training-free LR hybrid · DiffusionSfM uncertainty → MP-SfM depth-consistency gating · MegaSaM motion maps → MP-SfM BA pixel gating · RoMa v2 covariance → Pow3R depth prior.
- Notable: VGG-T3 identified as the natural bridge between feed-forward Tier-3 and Paradigm-D (mesh-in-one-pass) of [[gaussian-to-mesh-pipelines]].

## [2026-04-15] reingest-batch | Batch 5+6 — Pose estimation + radiance fields core (9 papers)
- Re-ingested: zhang2024_cameras-as-rays, barron2022_mip-nerf-360, barron2023_zip-nerf, park2023_camp, lin2024_vastgaussian, sun2025_sparse-voxels-rasterization, guo2025_ea-3dgs, kim2025_multiview-geometric-gs, tang2025_dronesplat
- Added Pipeline contribution sections to all 9.
- Thread augmented: radiance-field-evolution now carries Goal / 4 parallel Current SOTA pipelines (object-scale / city-scale / NeRF-family / neural-free) / lineage / candidate components / 3 synthesis bets / contradictions.
- Pipeline impact: radiance-field-evolution stack formalized in 4 regimes; city-scale three-way stack identified (VastGaussian partitioning / EA-3DGS compression / VPGS-SLAM incremental) as open synthesis bet.
- Synthesis bets proposed: port Mip-NeRF-360 distortion regularizer to 3DGS/SVRaster · stack VastGaussian+EA-3DGS+VPGS-SLAM at city scale · port CamP preconditioner to 3DGS pose-refinement · SVRaster primitives + CoMe confidence + MVS supervision.

## [2026-04-15] reingest-batch | Batch 7 — Gaussian → Mesh & SDF hybrids (10 papers)
- Re-ingested: chen2024_pgsr, gao2025_anisdf, guedon2025_milo, li2025_geosvr, li2025_va-gs, radl2025_sof, radl2026_confidence-mesh-3dgs, zhu2025_gs-discretized-sdf, kim2026_gauss-explorer, xie2025_gauss-mi
- Added Pipeline contribution sections to all 10.
- Pipeline impact: [[gaussian-to-mesh-pipelines]] Paradigm A stack fully enumerated (PGSR planar + VA-GS four-constraint + Kim 2025 MVS supervision + CoMe confidence + SOF meshing backend); Paradigm B = MILo; Paradigm C = GeoSVR / SVRaster; Paradigm D = VGG-T3. Inverse-rendering lane added (Zhu 2025 GS-SDF, AniSDF). Active-reconstruction (GauSS-MI) and VLM-reasoning (GaussExplorer) lanes added.
- Synthesis bets proposed: MILo Delaunay + CoMe confidence vertex-weighting · VA-GS $\mathcal{L}_f$ swapped to DINOv3 dense features · CoMe confidence as TSDF fusion weight · Zhu 2025 discrete-SDF + CoMe confidence unified uncertainty · GauSS-MI color-variance + CoMe confidence unified uncertainty model.

## [2026-04-15] reingest-batch | Batch 8 — Semantic / open-vocab 3D (8 papers, FINAL)
- Re-ingested: jatavallabhula2023_conceptfusion, qin2024_langsplat, ye2024_gaussian-grouping, bao2025_seg-wild, jiao2025_clip-gs, wu2026_langsvr, mao2025_spatiallm, chen2025_sam-3d
- Added Pipeline contribution to all 8.
- Thread augmented: lifting-foundation-models-to-3d now carries 8 parallel Current SOTA pipelines (Pipeline I–VIII: per-scene CLIP distillation / SAM identity / scene-level contrastive / voxel TSDF / one-stage sparse-voxel unified / VLM-reasoning / single-image generative / LLM-native structured), lineage, candidate components, four explicit synthesis bets, contradictions.
- Pipeline impact: the full open-vocab 3D landscape now mapped with axis-1 (what is lifted) and axis-2 (how it generalizes) concrete. Pipeline-II + SAM 3 native IDs flagged as highest-priority synthesis bet (delete Gaussian Grouping's association module).
- Synthesis bets proposed: Gaussian Grouping + SAM 3 identity IDs (kill cross-view association) · LangSplat + RADIOv2.5 + CoMe confidence · LangSVR + SAM 3 + DINOv3 + RADIOv2.5 full-upgrade · ConceptFusion + RADIOv2.5 + VPGS-SLAM online city-scale multimodal lifted 3D.

## [2026-04-15] reingest-complete | All 55 papers re-ingested
- 8 batches committed sequentially; zero papers remaining.
- Total Pipeline contribution sections written: 55.
- Threads retrofit with new SOTA-pipeline / lineage / candidate-components / synthesis-bets sections: 6 (foundation-features-for-geometry, open-vocab-2d-composition, gpu-native-sfm, gaussian-to-mesh-pipelines, radiance-field-evolution, lifting-foundation-models-to-3d).
- Threads left in original form (substantively cover the new template already): feed-forward-structure-from-motion, mono-depth-estimation, nerfstudio.
- Headline synthesis bets surfaced across the batch, flagged for experimental follow-up:
  1. Gaussian Grouping + SAM 3 native instance IDs → delete cross-view association module.
  2. CoMe confidence as TSDF fusion weight.
  3. City-scale three-way stack: VastGaussian partition + EA-3DGS codebook + VPGS-SLAM incremental submaps.
  4. TTT3R closed-form learning rate applied to RoMa v2 dense-match head.
  5. CamP preconditioner ported to 3DGS pose-refinement.
  6. RoMa v2 per-pixel covariance as InstantSfM Jacobian weighting.
  7. SVRaster primitives + CoMe confidence + MVS depth supervision as next-gen mesh pipeline.
  8. LoGeR SWA + TTT3R training-free learning rate hybrid for linear-time long-context feed-forward 3D.
  9. Unified per-primitive uncertainty model: CoMe confidence + GauSS-MI color-variance + Zhu-2025 discrete-SDF.
  10. LangSVR one-stage recipe with SAM 3 + DINOv3 + RADIOv2.5 — every 2024 component upgraded to 2026.
- Notable: "external-MVS-depth vs self-supervised-Gaussian-depth" contradiction flagged across both gaussian-to-mesh-pipelines and radiance-field-evolution remains the highest-leverage unresolved tension — principled switch is an open research direction, not yet addressed by any paper.
- Next operational step suggested: `lint frontmatter` to sweep `updated:` dates on the 55 touched paper pages + 6 threads.

## [2026-04-15] schema-change | Code + license tracking
- Updated: [CLAUDE.md](CLAUDE.md).
- §2 frontmatter: added `code:`, `license_paper:`, `license_code:`, `license_dataset:` fields with SPDX-first recording discipline and explicit handling of `unknown` / non-commercial / research-only values.
- §2 paper template: resource strip now includes a `[code]` link + paper/code license line; body gains an optional `## Code & license` section for restrictive-license callouts.
- §3.1 Step 0: new sub-steps 6 (find code — official or canonical community only; no guessing) and 7 (identify paper / code / dataset licenses from arXiv footer, LICENSE file, or dataset release page).
- §3.1 Step 3: paper-page writing now populates the resource + license frontmatter and body strip; pages without code record a `no code found (<date>)` marker so `lint find-code` has a timestamp.
- §3.3 bare lint: added diagnostics for "Missing code" (incl. 6-month-stale no-code markers), "Missing license", and "Restrictive licenses" (informational usability map).
- §3.3 lint actions: added `lint find-code` (re-searches for code on unlinked pages, refreshes stale markers) and `lint licenses` (fills paper / code / dataset license fields from arXiv, GitHub LICENSE, and dataset pages).
- §5 log format: added log shapes for `lint find-code` and `lint licenses`.
- Rationale: code availability and license restrictions materially affect whether a pipeline component is usable downstream — burying this in "open questions" of paper pages made it invisible. Treating licenses as first-class frontmatter lets designs ([[wiki/designs/]]) reason about redistributable stacks explicitly.
- Backfill needed: existing 55 paper pages have no `code:` / `license_*:` fields. First `lint find-code` + `lint licenses` pass will be substantive; run after this schema change settles.

## [2026-04-15] lint find-code
- Pages searched: 55 (54 papers + 1 dataset page wiki/datasets/tanks-and-temples.md). Plus 1 paper held for user decision (radl2026_confidence-mesh-3dgs — stub repo, code "coming soon") and 1 held for user decision (shi2026_self-distilled-roi — authors' repo but low-adoption medium-confidence).
- Newly linked: 47 pages with canonical code (top examples):
  - `wiki/papers/kirillov2023_sam.md` → github.com/facebookresearch/segment-anything [Apache-2.0]
  - `wiki/papers/schonberger2016_colmap-{sfm,mvs}.md` → github.com/colmap/colmap [BSD-3-Clause]
  - `wiki/papers/radford2021_clip.md` → github.com/openai/CLIP [MIT]
  - `wiki/papers/oquab2023_dinov2.md` → github.com/facebookresearch/dinov2 [Apache-2.0]
  - `wiki/papers/heinrich2025_radiov25.md` → github.com/NVlabs/RADIO [NSCL, non-commercial]
  - `wiki/papers/carion2026_sam-3.md` → github.com/facebookresearch/sam3 [SAM License]
  - `wiki/papers/chen2025_sam-3d.md` → github.com/facebookresearch/sam-3d-objects [SAM License]
  - `wiki/papers/qin2024_langsplat.md` → github.com/minghanqin/LangSplat
  - `wiki/papers/ye2024_gaussian-grouping.md` → github.com/lkeab/gaussian-grouping
  - `wiki/papers/wu2026_langsvr.md` → NONE (no code found)
- No-code-marker set (6): elflein2026_vgg-t3, jiao2025_clip-gs, kim2025_multiview-geometric-gs, kim2026_gauss-explorer, wu2026_langsvr, zhang2025_feed-forward-3d-survey.
- Restrictive-license callouts raised (10, written as `## Code & license` sections): carion2026_sam-3 (SAM License), chen2025_sam-3d (SAM License), heinrich2025_radiov25 (NSCL non-commercial), jang2025_pow3r (NAVER non-commercial), mao2025_spatiallm (Apache code + CC-BY-NC-4.0 weights), murai2025_mast3r-slam (DROID-SLAM-derived custom), radl2025_sof (NOASSERTION custom), sun2025_sparse-voxels-rasterization (NVIDIA Source Code License), yu2025_cusfm (NVIDIA Software License), zhong2026_instantsfm (NOASSERTION custom).
- Special-note callouts (7, written as `## Code & license` sections): curless1996_tsdf (Open3D is community impl of the concept, not paper-specific), edstedt2025_roma-v2 (MIT but DINOv3 components custom), jiao2025_clip-gs (name-collision warning: `gbliao/CLIP-GS` belongs to Liao 2404.14249, not Jiao 2412.19142), lin2024_vastgaussian (unofficial community reimplementation only, low confidence), simeoni2025_dinov3 (custom license, commercial-ok with conditions), zhang2025_feed-forward-3d-survey (survey, no dedicated code repo), zhang2025_loger (first-author's repo despite "reimplementation" naming).
- Not-addressed / held: radl2026_confidence-mesh-3dgs (stub repo), shi2026_self-distilled-roi (authors' repo, low-adoption). Await user decision before writing.
- License-field gap: ~20 pages recorded `license_code: unknown` because GitHub LICENSE file fetches were blocked during search — `lint licenses` follow-up will resolve these by reading the repos' LICENSE files directly.
- Total pages touched: 55 (all across 2 parallel edit-agents, 0 failures).

## [2026-04-15] lint licenses
- Filled: 56 paper-licenses, 23 code-licenses (`unknown` → resolved), 1 dataset-license.
- Paper licenses: 26 arxiv-nonexclusive (default), 12 CC-BY-4.0, 6 CC-BY-NC-SA-4.0, 2 CC-BY-SA-4.0, 2 CC-BY-NC-ND-4.0, 1 CVF-open-access (schonberger_colmap-sfm), 1 ACM (knapitsch_tanks-and-temples), 1 institutional (curless_tsdf), 1 unknown (chen_sam-3d — Meta project page, no explicit license).
- Code licenses resolved: 7 MIT, 4 Apache-2.0, 1 BSD-3-Clause, 1 GPL-3.0, 3 Gaussian-Splatting-License (langsplat / milo / ea-3dgs — inherits Inria 3DGS non-commercial terms), 1 NVIDIA-Source-Code-License (geosvr, inherits svraster), 1 CC-BY-NC-4.0 (discretized-sdf), 1 CC-BY-NC-SA-4.0 (ttt3r, inherits cut3r), 1 research-only (pgsr, ZJU custom), 3 `none` (chebbi_dense-matching, li_va-gs, zhang_loger — no LICENSE file, effectively all-rights-reserved under GitHub defaults).
- Dataset license: tanks-and-temples dataset → CC-BY-4.0 (toolbox already MIT).
- New `## Code & license` body sections added / updated: 9 ADD (chebbi, pgsr, ttt3r, milo, ea-3dgs, geosvr, langsplat, va-gs, discretized-sdf) + 2 UPDATE (vastgaussian — append Apache-2.0 confirmation, loger — append no-license warning).
- Unknowns retained: 1 paper license (chen_sam-3d — Meta project page has no explicit license line). Can be resolved if/when a companion arXiv version drops.
- Notable patterns: (1) Inria Gaussian-Splatting License propagates downstream — LangSplat / MILo / EA-3DGS all inherit 3DGS non-commercial restrictions, as does GeoSVR via SVRaster-NSCL and TTT3R via CUT3R. Any design that stacks these is structurally non-commercial regardless of the downstream license choice. (2) Paper licenses skew more open than code licenses — ~12 papers are CC-BY-4.0 but their matching repos are often restrictive or unlicensed. Classic academic pattern: open ideas, restricted implementations. (3) Three repos with no LICENSE file at all is a quieter but significant finding — these are effectively all-rights-reserved and block even forking.
- Total pages touched: 57 (56 papers + 1 dataset), 0 failures across 2 parallel edit-agents.

## [2026-04-15] design | Language-Grounded 3DGS — research synthesis + nerfstudio impl
- Created: [wiki/designs/language-grounded-3dgs-2026.md](wiki/designs/language-grounded-3dgs-2026.md) — research-synthesis design composing SAM 3 native IDs + RADIOv2.5 per-Gaussian distillation + LangSVR geometric-prior objective on a 3DGS substrate. Stage-by-stage pipeline (8 stages: mask source → feature source → geometric priors → per-Gaussian fields → one-stage loss → instance means → CSR identity hash → k-NN index), query/interaction layer (click/text/compositional/exemplar with decomposed latency budgets), densify/prune invariant rule set (the gap every prior paper punts on), deliberate non-goals, build sequence with falsifiable checkpoints.
- Created: [wiki/designs/language-grounded-3dgs-nerfstudio.md](wiki/designs/language-grounded-3dgs-nerfstudio.md) — 9-phase implementation plan (Phase 0 scaffolding → Phase 1 preprocessing CLI → Phase 2 identity field → Phase 3 language field → Phase 4 geometric priors → Phase 5 one-stage + full invariants → Phase 6 interaction-layer perf → Phase 7 editing ops → Phase 8 VLM escalation) into the visiofacto fork. Uses `VisiofactoLangModel(VisiofactoModel)` subclass + new config + `LanguageSceneDataParser`/`LanguageDataset` (following `SemanticDataset.get_metadata` pattern at `base_dataset.py:163-164`) + feature-channel trick from [[nerfstudio]] for rasterizing id_code / lang_latent. No upstream fork; no gsplat CUDA changes in Phases 0–5.
- Updated: [index.md](index.md) (Designs section now lists 3 designs; footer stats bumped)
- Pipeline impact: [[lifting-foundation-models-to-3d]] — proposed Pipeline IX (feature-augmented 3DGS with 2026-stack lifting) staged for promotion into SOTA pipelines section after verification. [[open-vocab-2d-composition]] — cross-thread design reference. Thread updates deferred to post-implementation (design calls for updating threads only after Build Sequence verification passes).
- Synthesis bet: composes three previously-filed synthesis bets from [[lifting-foundation-models-to-3d]] (SAM 3 replacing GG's association loop; RADIOv2.5 replacing CLIP+DINO+SAM composition; LangSVR-style one-stage + geometric priors), PLUS two new contributions no single paper provides: (1) densify/prune invariant rules with NULL_ID sentinel + boundary Gaussian protection, (2) precomputed interaction layer (CSR identity hash + per-instance mean latents + FAISS IVF-PQ fallback) hitting click <5 ms + text <100 ms at 1 M Gaussians.
- Notable:
  - Design explicitly ports LangSVR's objective function from sparse voxels to 3DGS while rejecting LangSVR's substrate claim — the first on-paper argument for that split.
  - Identity field sized at 8-D continuous + 14-bit VQ (16 384 codebook); language latent at 32-D (pushes back against LangSplat's "low-D unspecified" and Gaussian Grouping's unprincipled identity-dim hyperparameter).
  - Non-commercial-license stack (SAM 3, RADIOv2.5, LangSplat) flagged as downstream-use blocker but not design blocker; swap path documented.
  - Implementation plan includes 9-phase breakdown with M1–M6 milestone gates so project has natural stopping points if resources run out before M6.

## [2026-04-15] cascade | Pipeline IX filed as candidate in lifting-FM-to-3D thread
- Updated: [wiki/threads/lifting-foundation-models-to-3d.md](wiki/threads/lifting-foundation-models-to-3d.md) — added Pipeline IX proposal to "Candidate components / not yet integrated" (composes SAM 3 + RADIOv2.5 + DINOv3 + LangSplat-style AE, plus the design-specific densify/prune invariant + precomputed interaction layer). Added [wiki/designs/language-grounded-3dgs-2026.md](wiki/designs/language-grounded-3dgs-2026.md) to sources: frontmatter.
- Notable: entry explicitly labeled "not promoted to Current SOTA pipelines — until Build-Sequence verification passes" per Hard Rule §6.3. The 8 existing SOTA pipelines in the thread remain unchanged; Pipeline IX sits in candidates until evidence moves it.

## [2026-04-18] schema-migration | Forward-only migration to the new CLAUDE.md schema
- Goal: roll existing wiki content forward to the schema upgrade that landed earlier today (typed Goal + optional structured contract, Pareto operating points capped at 3, capability gaps section, bet ranking rubric, design closure loop, assumption conflict registry, first-class idea + stage pages, retired `lint extract-ideas`).
- Step 1 — Thread goal contracts: added `## Goal` + `## Goal contract` YAML blocks to all 9 threads. `nerfstudio.md` keeps a deliberately empty contract (reference-codebase thread, not a pipeline thread).
- Step 2 — Operating-point migration: set `operating_points:` frontmatter on all 9 threads. Single-OP (`op:default`) for `mono-depth-estimation`, `foundation-features-for-geometry`, `feed-forward-structure-from-motion`, `nerfstudio`. Two OPs for `gpu-native-sfm` (`op:general-purpose`, `op:sequential-slam-prior`). Three OPs for `open-vocab-2d-composition`, `gaussian-to-mesh-pipelines`, `radiance-field-evolution`, `lifting-foundation-models-to-3d`. `radiance-field-evolution` Zip-NeRF stack demoted from its own OP to `## Candidate components` (NeRF-family ceiling reference, still load-bearing but no current 3DGS paper moves it). `lifting-foundation-models-to-3d` consolidated 8 prior "Pipelines" into 3 OPs and flagged Pipelines VI–VIII (VLM-reasoning, generative-3D-from-2D, LLM-native-structured) for **future thread split** — parked in candidates under a migration note until sibling threads are created.
- Step 3 — Capability-gaps seeding: added `## Capability gaps` to all 9 threads, populated from shelved-bet triggers and unmet upstream properties where applicable.
- Step 4 — Bet numbering + rubric backfill: 20 existing prose bets converted to structured `Bet #001`…`Bet #020` entries with `status`, `combines`, `stage_target`, `op_target`, `confidence`, `magnitude`, `cost`, `breakage_risk`, `hypothesis`, `expected_gain`, `risk`, `validating_experiment`, `triggers` fields. Bet #020 newly minted from the `lifting-foundation-models-to-3d` Pipeline IX candidate so the existing design docs have a bet to point at.
- Step 5 — Design closure backfill: added `realizes_bet`, `realizes_ideas`, `outcome`, `outcome_date` to all 4 designs. `language-grounded-3dgs-2026` and `language-grounded-3dgs-nerfstudio` point at **Bet #020**. `come-integration-nerfstudio` and `reingest-tracker` are marked `realizes_bet: none` — the first is a direct paper re-implementation, the second a meta-workflow tracker. All four designs `outcome: pending`.
- Step 6 — Idea back-fill (partial coverage, complete for bet-referenced papers): created 36 idea pages in `wiki/ideas/` and 3 stage pages in `wiki/stages/`. Covered: CoMe, Gaussian Grouping, SAM 3, LangSplat, InstantSfM, VastGaussian, DINOv3, RADIOv2.5, SVRaster, COLMAP-MVS, RoMa v2, EA-3DGS, VPGS-SLAM, CamP, MILo, PGSR, VA-GS, GeoSVR, Kim-2025-multiview-geometric-GS, CuSfM, MP-SfM, TTT3R, LangSVR, ConceptFusion, Trident, SD-RPN. Every `combines:` slug referenced in Bets #001–#020 now has a corresponding `wiki/ideas/<slug>.md`.
- Step 6 remaining (29 papers): barron2022_mip-nerf-360, barron2023_zip-nerf, bao2025_seg-wild, chebbi2025_multiview-dense-matching, chen2025_sam-3d, curless1996_tsdf, elflein2026_vgg-t3, gao2025_anisdf, jang2025_pow3r, jiao2025_clip-gs, jin2026_zipmap, kim2026_gauss-explorer, kirillov2023_sam, knapitsch2017_tanks-and-temples (dataset — does not need idea pages), li2025_megasam, mao2025_spatiallm, murai2025_mast3r-slam, oquab2023_dinov2, pan2024_glomap, radford2021_clip, radl2025_sof, schonberger2016_colmap-sfm, tang2025_dronesplat, xie2025_gauss-mi, yao2018_mvsnet, zhang2024_cameras-as-rays, zhang2025_feed-forward-3d-survey (survey — low priority), zhang2025_loger, zhao2025_diffusionsfm, zhu2025_gs-discretized-sdf. These fall into three categories: (a) foundational papers whose concepts already exist as `wiki/methods/` pages (CLIP, SAM, DINOv2, TSDF, COLMAP, MVSNet); (b) survey / dataset papers not needing idea pages; (c) papers not currently referenced in any numbered bet. They can be processed in a follow-up migration run or during normal ingests.
- Stage pages: 3 created (`radiance-fields.loss-balancing`, `radiance-fields.regularization`, `radiance-fields.appearance-compensation`). ~40 more stage slugs are referenced in idea frontmatter but have no pages yet — the schema mandates stage pages be created **opportunistically** when a thread references them, so this is a known follow-up and not a lint violation yet.
- Manual-follow-up flags: `lifting-foundation-models-to-3d` thread split (Pipelines VI–VIII into sibling threads); non-commercial licenses on many 3DGS-stack papers (CoMe, LangSplat, SVRaster, GeoSVR, MILo, PGSR, EA-3DGS, RADIOv2.5) — any commercial downstream pipeline must re-validate licenses; stage-page creation batch (~40 stubs) when threads start consuming stage wikilinks directly.
- Not touched: paper page bodies (no re-ingest this round — ideas are extracted from existing `## Pipeline contribution` sections). Bet lineage entries in thread `## Pipeline lineage` sections not retroactively prefixed with `op:<name>` for every historical entry — only new entries are prefixed, old entries left as-is to avoid rewriting history.

## [2026-04-18] schema-migration-followup | Complete the 2026-04-18 migration — 4 remaining phases
- Phase 1 (idea back-fill): 48 new idea pages created, covering every previously-unprocessed paper except the 2 intentional skips (`knapitsch2017_tanks-and-temples` — dataset paper; `zhang2025_feed-forward-3d-survey` — survey paper). Total idea pages across the wiki: 74 (was 36 after the first migration pass). Every `combines:` slug referenced in Bets #001–#025 now resolves to an existing idea page.
- Phase 2 (thread split): 3 new sibling threads created from `lifting-foundation-models-to-3d`'s parked Pipelines VI–VIII: [wiki/threads/vlm-reasoning-over-3d-scenes.md](wiki/threads/vlm-reasoning-over-3d-scenes.md) (from GaussExplorer), [wiki/threads/generative-3d-from-2d-priors.md](wiki/threads/generative-3d-from-2d-priors.md) (from SAM 3D), [wiki/threads/llm-native-structured-scenes.md](wiki/threads/llm-native-structured-scenes.md) (from SpatialLM). Parent thread's parked-pipelines section replaced with sibling-thread cross-links; superseded lineage entries consolidated into one topology-change entry. Total threads across the wiki: 12. Five new synthesis bets (Bets #021–#025) minted in the sibling threads, connecting them back to the parent thread's OPs via cross-thread compositions.
- Phase 3 (stage stubs): 84 stage pages created organized by domain — radiance-fields (16), sfm (12), feed-forward-sfm (15), lifting-foundation-models (12), open-vocab-2d (5), mvs (4), mesh-reconstruction (2), feature-matching (2), foundation-features (1), vlm-reasoning (3), generative-3d (3), llm-structured-scenes (3), relighting (1), active-reconstruction (2). `grep`-verified: zero stage-slug references in idea frontmatter resolve to a missing page. Two `alias` stages created (`radiance-fields.primitive-representation` → `primitives`, `feed-forward-sfm.pose-recovery` → `pose-estimation`, `feed-forward-sfm.attention-mechanism` → `global-attention`) to absorb inconsistent slugs without rewriting idea pages.
- Phase 4 (license audit): [wiki/meta/license-audit.md](wiki/meta/license-audit.md) published. Summary: 27 commercial-OK papers (MIT/Apache-2.0/BSD/GPL-3.0), 21 restricted-code papers (SAM License, NVIDIA NSCL, Gaussian-Splatting-License, research-only, NOASSERTION), bet-level readiness table flags **8 of 20 bets as fully blocked** in a commercial deployment until the restricted components are re-implemented / licensed / swapped. Key commercial blockers: SAM 3 (SAM License), RADIOv2.5 (NVIDIA NSCL), all Gaussian-Splatting-License-derived code (CoMe, LangSplat, MILo, PGSR, GeoSVR, EA-3DGS, LangSVR). Remediation checklist attached.
- `index.md` updated: 12 threads (9 + 3 new), 74 ideas + 84 stages + 1 meta referenced. Per-idea and per-stage lines intentionally omitted to keep the index under the 300-line split threshold; browse `wiki/ideas/` and `wiki/stages/` directly.
- Migration prompt `/home/wii/.claude/prompts/migrate-wiki-to-new-schema.md` is now fully executed. The user may delete it.
- Pipeline impact: no thread's Current SOTA pipeline was changed in this pass (this pass is schema hygiene, not synthesis). Pass B was not run — that's triggered by new ingests, not back-fill.
- Notable: the two `alias` stages expose a small canonicalization debt — future ingests should prefer the canonical slug and the alias stubs can be deleted once no idea references them. Likewise, 3 restricted licenses (DINOv3 License, InstantSfM NOASSERTION, CuSfM NOASSERTION) are flagged in the audit for manual review — they *may* be commercial-OK under their specific terms, which would un-block several bets.

## [2026-04-18] schema-change | Licenses are not bet-blockers (§6.15 added)
- **Policy.** Non-commercial / research-only / unknown licenses never block a bet or lower a bet's `confidence`, and never remove an idea from a thread's SOTA pipeline. Priority is always the strongest pipeline mechanism-level reasoning can assemble. Licenses are implementation-time concerns handled at build time via {separate-license, re-implement, swap}.
- **CLAUDE.md changes.** Added Hard Rule §6.15 stating the policy explicitly; removed the `commercial-license-ok` ↔ `requires-non-commercial-component` pair from §7 Assumption conflict registry; amended §3.1 Step 5 Pass A "Assumption check" to note license tokens are skipped; amended `lint type-check` description with the same exemption.
- **License audit rewrite.** [wiki/meta/license-audit.md](wiki/meta/license-audit.md) no longer contains any "Blocked" or "OK" verdicts at the bet level. The bet-level table is now "Bet-level implementation-time license annotations" — every bet stands; each row recommends a cheapest implementation path at build time. Section 6 renamed from "Remediation checklist" to "Implementation-time checklist" and reframed as build-time tasks, not pre-adoption gates.
- **Bet state change.** None. All 25 bets (#001–#025) keep their existing `status`, `confidence`, `magnitude`, `cost`, `breakage_risk`. The earlier license-audit text that called 8 bets "blocked" was never reflected in the bet frontmatter itself, so this is a documentation consistency fix, not a state rollback.
- **Pipeline impact.** No thread's SOTA pipeline composition changed. Non-commercial-licensed ideas (CoMe, LangSplat, Gaussian Grouping, SAM 3, RADIOv2.5, MILo, PGSR, VA-GS, GeoSVR, EA-3DGS, LangSVR, SVRaster) stay where they were.
- **Notable.** The policy keeps the wiki's research-synthesis mandate clean — bet discovery is mechanism-level reasoning, license resolution is an engineering task that happens later and is fungible. Flagging restricted licenses during ingest remains required (Step 0.7) as useful planning info; they just don't veto anything upstream.

## [2026-04-21] ingest | MP-SfM (Pataki 2025) — PDF cache-fill + deep re-read
- local_paper: papers/sfm-slam/pataki_2025_mp-sfm.pdf (moved from raw/, 7.7 MB)
- Context: paper + lumped idea pages already existed from a prior shallow ingest (online reading, no PDF cached). Full PDF read triggered atomic-contribution enumeration the first pass missed.
- Created stage pages: [wiki/stages/sfm.mono-depth-lifted-registration.md](wiki/stages/sfm.mono-depth-lifted-registration.md), [wiki/stages/sfm.depth-constrained-ba.md](wiki/stages/sfm.depth-constrained-ba.md), [wiki/stages/sfm.forward-backward-depth-consistency.md](wiki/stages/sfm.forward-backward-depth-consistency.md). These were declared in the existing lumped idea's `rewrites.introduces:` but had no pages — schema-integrity gap closed (Hard Rule §6.12).
- Created ideas: [wiki/ideas/matcher-score-next-view-selection_pataki2025.md](wiki/ideas/matcher-score-next-view-selection_pataki2025.md) (A2, `stage-swap`), [wiki/ideas/depth-proportional-uncertainty-fusion_pataki2025.md](wiki/ideas/depth-proportional-uncertainty-fusion_pataki2025.md) (A3, `drop-in`), [wiki/ideas/bilateral-normal-integration-with-uncertainty_pataki2025.md](wiki/ideas/bilateral-normal-integration-with-uncertainty_pataki2025.md) (A4, `stage-swap` sub-mechanism). Each is independently reusable across pipelines, not MP-SfM-specific.
- Updated lumped idea: [wiki/ideas/mono-depth-normal-constrained-incremental-sfm_pataki2025.md](wiki/ideas/mono-depth-normal-constrained-incremental-sfm_pataki2025.md) — added `co_requires: [A2, A3, A4]` (bundle truth — ablations show these are not optional inside MP-SfM); expanded Mechanism with the two-view-init mono-depth fallback (§3.1) and the per-view median-ratio depth rescaling (Eq. 1); added "Trade-offs vs. the decomposed pipeline" section per topology-rewrite hygiene.
- Updated paper page: [wiki/papers/pataki2025_mp-sfm.md](wiki/papers/pataki2025_mp-sfm.md) — Method section rewritten as five coupled sub-mechanisms + uncertainty calibration; Pipeline-contribution section now lists the four atomic ideas with wikilinks and candidate thread targets.
- Pipeline impact:
  - [[feed-forward-structure-from-motion]] (Tier-2 survey thread): updated MP-SfM bullet to reference all four atomic ideas; refined the "Principled fusion of ≥3 learned priors" capability gap with the now-separable recipes.
  - [[gpu-native-sfm]]: refined **Bet #010** — `combines:` tightened from three paper-links to four atomic-idea-links ([[depth-constrained-jacobian_zhong2026]], [[bilateral-normal-integration-with-uncertainty_pataki2025]], [[depth-proportional-uncertainty-fusion_pataki2025]], [[roma-v2-predictive-covariance_edstedt2025]]); `stage_target` updated to [[sfm.depth-constrained-ba]]. Filed **Bet #013** (Port matcher-score next-view selection into Tier-1 GPU SfM) — `combines: [[matcher-score-next-view-selection_pataki2025]], [[depth-constrained-jacobian_zhong2026]]`, hours-cost low-risk drop-in.
  - [[mono-depth-estimation]]: added A3 as a `drop-in` answer to the "calibrated per-pixel uncertainty" capability gap; updated SfM-prior evidence entry + open-question on prior-fusion.
- Pass B cross-thread transfer: filed **Bet #004** on [[radiance-field-evolution]] — use A3's uncertainty-calibration recipe as per-pixel inverse-variance weight on 3DGS depth-supervision losses ([[median-depth-relative-loss_kim2025]], [[va-gs-four-loss-stack_li2025]]). None of the source papers proposes this composition; A3's `drop-in` scope makes it low-cost to validate.
- Notable: the *lumping vs. splitting* decision for atomic ideas is recorded on-page via `co_requires:` — the `topology-rewrite` stays as the system-level idea (what "adopting MP-SfM" means for a pipeline), while A2/A3/A4 are independently adoptable atoms that compose back into the system via the bundle edge. This matches `§1.6` guidance about bundle truth being a property of the idea.
- Not created (deferred): Metric3Dv2, Cao-2022 bilateral-normal-integration, DSINE, Doppelgangers — each load-bearing but referenced only 1–3 times; future ingests of those source papers will create them under `§3.1 Step 4` cascade rules.
- `raw/` now empty.

## [2026-04-21] ingest | DetectorFreeSfM (He 2023) + MADPose (Yu 2025) — batch of 2 PDFs

Two papers ingested together from `raw/`: both touch the SfM / pose-estimation stack from orthogonal angles (detector-free matching frontend vs. depth-aware pair-pose). `raw/` emptied (was 2 arxiv-named PDFs).

- **local_paper:** papers/sfm-slam/he_2023_detector-free-sfm.pdf (21.5 MB, Apache-2.0 code, arxiv-nonexclusive paper); papers/pose-estimation/yu_2025_madpose.pdf (25.8 MB, BSD-3-Clause code, arxiv-nonexclusive paper). No commercial-use blockers in either.

- **Created papers:** [wiki/papers/he2023_detector-free-sfm.md](wiki/papers/he2023_detector-free-sfm.md), [wiki/papers/yu2025_madpose.md](wiki/papers/yu2025_madpose.md). Plus three stubs referenced by both: [wiki/papers/sun2021_loftr.md](wiki/papers/sun2021_loftr.md) (detector-free matcher; Apache-2.0), [wiki/papers/lindenberger2021_pixsfm.md](wiki/papers/lindenberger2021_pixsfm.md) (DetectorFreeSfM's direct baseline; Apache-2.0), [wiki/papers/wang2025_moge.md](wiki/papers/wang2025_moge.md) (MADPose's best MDE; MIT).

- **Created ideas (6, two bundles):**
  - DetectorFreeSfM bundle (`co_requires:` all three): [wiki/ideas/coarse-to-fine-detector-free-sfm-bridge_he2023.md](wiki/ideas/coarse-to-fine-detector-free-sfm-bridge_he2023.md) (`scope: bridge`, retypes detector-free matches → classical SfM input via coarse-stride quantization); [wiki/ideas/multi-view-transformer-track-refinement_he2023.md](wiki/ideas/multi-view-transformer-track-refinement_he2023.md) (`scope: stage-swap`, transformer multi-view refinement with `w×w` ref-location search); [wiki/ideas/iterative-ba-plus-track-topology-adjustment_he2023.md](wiki/ideas/iterative-ba-plus-track-topology-adjustment_he2023.md) (`scope: stage-split`, introduces [[sfm.track-topology-adjustment]] alongside iterative-triangulation-BA).
  - MADPose bundle (hybrid estimator `co_requires:` the two atoms): [wiki/ideas/affine-corrected-minimal-relative-pose-solvers_yu2025.md](wiki/ideas/affine-corrected-minimal-relative-pose-solvers_yu2025.md) (`scope: stage-swap`, calibrated 3pt / shared-focal 4pt / two-focal 4pt Gröbner-basis solvers for pose + α + β₁ + β₂ + focals); [wiki/ideas/depth-induced-reprojection-scoring_yu2025.md](wiki/ideas/depth-induced-reprojection-scoring_yu2025.md) (`scope: drop-in`, symmetric truncated reprojection under solved affine corrections); [wiki/ideas/hybrid-lo-msac-dual-modality-estimator_yu2025.md](wiki/ideas/hybrid-lo-msac-dual-modality-estimator_yu2025.md) (`scope: stage-swap`, bundle wrapper alternating depth-aware + point-based solvers with dual scoring).

- **Created stages (5):** [wiki/stages/sfm.feature-track-refinement.md](wiki/stages/sfm.feature-track-refinement.md), [wiki/stages/sfm.track-topology-adjustment.md](wiki/stages/sfm.track-topology-adjustment.md), [wiki/stages/pose-estimation.relative-pose-solver.md](wiki/stages/pose-estimation.relative-pose-solver.md), [wiki/stages/pose-estimation.robust-estimator-scoring.md](wiki/stages/pose-estimation.robust-estimator-scoring.md), [wiki/stages/pose-estimation.hybrid-robust-estimator.md](wiki/stages/pose-estimation.hybrid-robust-estimator.md). New `pose-estimation.*` domain registered (three stages) — previously the wiki handled relative-pose only via `feed-forward-sfm.pose-estimation`, which is SfM-internal.

- **Created thread:** [wiki/threads/relative-pose-estimation.md](wiki/threads/relative-pose-estimation.md) — new thread per §3.3 goal-first workflow, `op:default`, `status: draft` pending user review of Goal + contract. SOTA pipeline is the MADPose bundle node (n3 hybrid estimator `co_requires:` n4 solver + n5 scoring). Three bets filed: #014 (MADPose 3pt solver as MP-SfM two-view init), #015 (depth-induced reprojection as third model in COLMAP geometric verification), #016 (MADPose pair-pose frontend for InstantSfM global positioning — cross-filed in gpu-native-sfm).

- **Pipeline impact:**
  - [[feed-forward-structure-from-motion]] · op:default: Pass A added DetectorFreeSfM + MADPose to Tier 2 Evidence; capability gaps updated (texture-poor + low-overlap intersection now addressable; pair-pose-as-SfM-frontend gap opened). Pass B filed **Bet #019** (DetectorFreeSfM × MP-SfM compose — texture-poor AND low-overlap simultaneously) and **Bet #020** (DINOv3-backboned RoMa v2 → DetectorFreeSfM frontend swap).
  - [[gpu-native-sfm]] · op:general-purpose: Pass A resolved the long-standing "Learned feature frontend" candidate (DetectorFreeSfM is the clean benchmark — 340× less BA memory than PixSfM); added three new Candidate-component entries (DetectorFreeSfM track-refinement + topology-adjustment; MADPose hybrid estimator as pair-pose frontend; depth-induced reprojection in geometric verification). Pass B filed **Bet #017** (DetectorFreeSfM frontend → InstantSfM backend for texture-poor GPU-native SfM). Capability gaps refreshed. Related-threads link to new [[relative-pose-estimation]] added.
  - [[mono-depth-estimation]] · op:default: Pass A added a new **"As relative-pose prior"** evidence section (MADPose is the first wiki paper consuming mono-depth at the pair-pose stage, not multi-view BA). Added MoGe. Capability gaps expanded: cross-pair shift consistency, metric-depth-that's-actually-metric. Pass B note logged (no bets filed here — mono-depth is consumed by others; bets live where consumption happens).
  - [[relative-pose-estimation]] (new): SOTA pipeline built from MADPose; three bets (#014, #015, #016) open.

- **Pass B cross-paper synthesis (6 novel bets, none proposed in any single paper):**
  - **#014** — MADPose 3pt solver replaces MP-SfM's median-ratio depth rescaling (Eq. 1) in two-view bootstrap. Filed in [[relative-pose-estimation]].
  - **#015** — depth-induced reprojection as third model (H + F + E_r) in COLMAP multi-model geometric verification. Filed in [[relative-pose-estimation]].
  - **#016** — MADPose hybrid estimator as pair-pose frontend for InstantSfM / GLOMAP global positioning. Cross-filed in [[relative-pose-estimation]] and [[gpu-native-sfm]].
  - **#017** — DetectorFreeSfM frontend (LoFTR + quantized bridge + transformer refinement + BA/TA loop) plugged into InstantSfM's GPU-native backend. Filed in [[gpu-native-sfm]].
  - **#019** — DetectorFreeSfM × MP-SfM combo: attacks texture-poor + low-overlap jointly, not tested in either source paper. Filed in [[feed-forward-structure-from-motion]].
  - **#020** — LoFTR → RoMa v2 on DINOv3 swap inside DetectorFreeSfM bridge; not tested in source (2021–2022 matchers only). Filed in [[feed-forward-structure-from-motion]].

- **Notable:**
  - **Bundle truth scales across batches.** Both papers produce multi-idea bundles (`co_requires:`). Bets that combine ideas across bundles (e.g. #017 needs all three DetectorFreeSfM ideas plus one InstantSfM idea) are explicit about which bundles must adopt. This matches §1.6 / §6.14.
  - **Scope-integrity calls:** idea A1 (match-quantization) classified as `bridge`, not `stage-swap`, because it retypes a stage boundary rather than replacing a stage filler. Idea A3 (BA+TA) classified as `stage-split` rather than `stage-swap`, because the paper ablates TA as a separate 4.6%-contributing operation with its own stage [[sfm.track-topology-adjustment]]. Coercing either into `stage-swap` would hide the pipeline-shape implications.
  - **Empirical surprise from MADPose Fig. 6**: fitted per-view `β` > 10% of median depth is routine even for *metric*-trained MDE models. Recorded as a Contradiction in [[relative-pose-estimation]] and as a new Capability gap in [[mono-depth-estimation]] — shift modeling is load-bearing everywhere mono-depth feeds geometry, not a corner case.
  - **License hygiene:** DetectorFreeSfM Apache-2.0 + MADPose BSD-3-Clause + LoFTR Apache-2.0 + PixSfM Apache-2.0 + MoGe MIT — all fully commercial-OK. No additions to license-audit restricted list; per §6.15 this did not affect any bet `confidence` / `status`.
  - **New thread §3.3 note:** [[relative-pose-estimation]] created with `status: stable`. Goal + contract + OPs drafted from the seed papers; will promote past `draft` on user approval.

- **Not created (deferred):** Barath 2022 2pt+D, Ding 2024 3p3d/4p4d, Nistér 2003 5pt, Stewénius 2008 6pt, Camposeco 2018 hybrid RANSAC, Lebeda 2012 LO-RANSAC, Larsson 2017 Gröbner-basis template, AspanTransformer, MatchFormer, S2DNet, Depth-Anything v1/v2, Marigold, Omnidata, RansacLib, PoseLib — each referenced by one paper only or deep lineage; cascade stubs deferred to §3.1 Step 4 until a future ingest relies on their mechanism.

- `raw/` now empty.

## [2026-04-21] ingest | MASt3R (Leroy 2024) — Grounding Image Matching in 3D

Single-paper ingest from `raw/` (19MB arxiv-named PDF `2406.09756v1.pdf`). MASt3R is the direct descendant of DUSt3R and the matching-aware Tier-3 anchor of the feed-forward reconstruction lineage; its ingest closes a gap in the wiki where MASt3R was referenced by seven downstream papers (MASt3R-SLAM, MP-SfM, PoW3R, RoMa v2, SpatialLM, etc.) but had no paper page of its own.

- **local_paper:** papers/feature-matching/leroy_2024_mast3r.pdf (19 MB). Code: [github.com/naver/mast3r](https://github.com/naver/mast3r) CC-BY-NC-SA-4.0 (non-commercial), weights same license plus per-dataset constraints (Map-free dataset license is particularly restrictive). Paper: arxiv-nonexclusive. **Commercial-use blocker** flagged on the paper page under §6.15 — informational only; does not affect bet confidence.

- **Created paper:** [wiki/papers/leroy2024_mast3r.md](wiki/papers/leroy2024_mast3r.md).

- **Created ideas (4 atomic, all independently reusable, no `co_requires:` bundle):**
  - [wiki/ideas/dust3r-matching-head_leroy2024.md](wiki/ideas/dust3r-matching-head_leroy2024.md) — `scope: stage-swap` at [[feed-forward-sfm.frame-matching]]. Dual-head DUSt3R with a 2-layer MLP+GELU descriptor head trained via InfoNCE over GT 3D correspondences. Load-bearing ablation: Table 1 III vs IV — match-only 10.8° median rot vs joint 3.0°, confirming bidirectional synergy between 3D regression and matching supervision.
  - [wiki/ideas/metric-scale-pointmap-loss_leroy2024.md](wiki/ideas/metric-scale-pointmap-loss_leroy2024.md) — `scope: drop-in` at [[feed-forward-sfm.geometry-output]]. Sets `z := ẑ` in DUSt3R's regression loss on the metric-GT training subset, so the network regresses absolute scale without new parameters. Load-bearing ablation: Table 1 IV vs V — swapping DPT depth for MASt3R's own metric pointmap jumps VCRE AUC from 0.752 → 0.934 and cuts median translation error from 0.93m → 0.46m, the largest single ablation in the paper.
  - [wiki/ideas/fast-reciprocal-nn-matching_leroy2024.md](wiki/ideas/fast-reciprocal-nn-matching_leroy2024.md) — `scope: drop-in` at new stage [[feature-matching.reciprocal-matching]]. Iterative cycle-finder, `O(k·WH)` vs `O(W²H²)`. Counter-intuitive ablation: performance *improves* as `k` decreases from `all` to ~3000 (64× CPU speedup + higher AUC), explained by cycle-consistency acting as implicit outlier filter.
  - [wiki/ideas/coarse-to-fine-window-covering-matching_leroy2024.md](wiki/ideas/coarse-to-fine-window-covering-matching_leroy2024.md) — `scope: drop-in` wrapper at [[feed-forward-sfm.frame-matching]]. Greedy set-cover of full-resolution window pairs seeded by a 512px coarse pass; principal-point preserved under homography crops. 4.6× DTU zero-shot Chamfer reduction over DUSt3R. Evidence is less clean — no standalone ablation.

- **Created stages (1):** [wiki/stages/feature-matching.reciprocal-matching.md](wiki/stages/feature-matching.reciprocal-matching.md) — dense descriptor pair → reciprocal-correspondence set with `provides_properties: [cycle-consistency, outlier-filtered-matches]`. Registers the feature-matching subsystem's third stage page.

- **Updated methods:** [wiki/methods/mast3r.md](wiki/methods/mast3r.md) promoted stub → draft with mechanism detail, lineage to DUSt3R, downstream consumer list. [wiki/methods/dust3r.md](wiki/methods/dust3r.md) gained a "Direct successor" section + MASt3R in sources/references.

- **Pipeline impact:**
  - [[feed-forward-structure-from-motion]] · op:default: Pass A added MASt3R as a first-class Tier-3 entry (previously only referenced indirectly via MASt3R-SLAM). **Capability gap "calibration-grade outdoor accuracy from fully feed-forward methods" partially closed at the pair-pose level** — Idea B achieves 94.1 VCRE AUC on Map-free from a single forward pass (no matching, no RANSAC). Remaining gap: sequence-level metric consistency. Three Pass B bets filed.
  - [[relative-pose-estimation]] · op:default: Pass A formalized MASt3R matcher reference in n1 with explicit [[dust3r-matching-head_leroy2024]] wikilink. Two Candidate-component entries added: MASt3R direct-regression pose (94.1 AUC Map-free via metric pointmap + PnP, brittle to calibration mismatch) and fast-reciprocal-NN as matcher-agnostic speedup. No new bets here — existing #014–#016 already cover the depth-aware-pose synthesis space; MASt3R-specific bets live in [[feed-forward-structure-from-motion]] where the sequence-level story belongs.
  - [[foundation-features-for-geometry]] · op:default: capability gap "calibration-grade feed-forward poses without classical BA" updated — partially closed by MASt3R (Idea B) *but* using CroCo/DUSt3R pretraining, not DINOv3. Evidence that "foundation features" is broader than "frozen DINOv3" for geometry; 3D-task-pretrained encoders are a parallel design point. Dense-matching evidence section extended with MASt3R as a contrast to RoMa v2 (joint 3D + descriptor training vs frozen-backbone + matching head).

- **Pass B cross-paper synthesis (3 novel bets, none proposed in source paper):**
  - **#026** (in [[feed-forward-structure-from-motion]]) — MASt3R metric pointmap as sequence-level Sim(3) anchor for TTT3R's closed-form LR; targets the remaining "sequence-level calibration-grade" portion of the capability gap.
  - **#027** (in [[feed-forward-structure-from-motion]]) — Fast reciprocal-NN algorithm on RoMa v2's dense features; Idea C is matcher-agnostic and could replace RoMa v2's CUDA refinement step with implicit outlier filtering.
  - **#028** (in [[feed-forward-structure-from-motion]]) — MASt3R direct-regression pose as MP-SfM two-view bootstrap; distinct from Bet #014 (which uses MADPose's 3pt solver on unstructured mono-depth); this bet uses MASt3R's absolute-scale pointmap directly, avoiding alignment to a triangulated subgraph entirely.

- **Notable:**
  - **Four independent atoms.** Unlike the DetectorFreeSfM / MADPose ingest which produced `co_requires:` bundles, all four MASt3R ideas are independently reusable. The paper combines them, but a thread can adopt Idea C (fast reciprocal) with RoMa v2 without adopting Idea A (dual-head), or adopt Idea B (metric scale) without the descriptor head. This shapes the Pass B bet space — more combinatorial freedom, more cross-thread transfer opportunities.
  - **Scope classification — judgment call on Idea A.** Classified `stage-swap` (not `multi-stage-collapse` or `new-paradigm`): MASt3R produces the same output type as [[dust3r|DUSt3R]] at the same stage, just with a stronger mechanism. Coercing the match head into a new stage would have over-stated the pipeline-shape implications; keeping it as a stage-swap honors that MASt3R uses DUSt3R's architecture and checkpoint as the initialization.
  - **Idea C and the "fewer = better" monotonicity.** The counter-intuitive AUC-increases-as-k-decreases result (Fig. 3 right) is the strongest mechanistic insight in the paper and is the common generalization point — if cycle-consistency on sub-sampled dense matchers implicitly filters outliers, this should transplant to any dense matcher's reciprocal-NN step. Bet #027 is the direct test.
  - **Evidence asymmetry.** Ideas A, B, C each have clean standalone ablations isolating the contribution. Idea D (coarse-to-fine) doesn't — it's inferred from DTU MVS gains (4.6× Chamfer reduction) and from the fact that Aachen at 1600×1200 requires it. Flagged in the idea page's "Why it wins" section per §3.1 Step 2.
  - **License hygiene.** MASt3R's CC-BY-NC-SA-4.0 is the second restrictive license in the wiki (after Gaussian Splatting's custom license). Worth a note in the License Audit meta page, but per §6.15 the license does not block Bets #026–#028's adoption, confidence, or SOTA composition — those are mechanism-only decisions. Implementation-time commercial plans may need retraining on a commercial-safe data mix.
  - **Downstream consumer density.** MASt3R is referenced by 8 existing wiki pages (MASt3R-SLAM paper, MP-SfM paper, PoW3R paper, RoMa v2 paper, ZipMap paper, TTT3R paper, method pages for DUSt3R + MASt3R itself, and feed-forward-SfM thread). Pre-ingest, all of these cited "MASt3R" without a backing paper page. The ingest now closes that referential debt.

- **Not created (deferred):** CroCo (pretraining predecessor; MASt3R uses DUSt3R's checkpoint which inherits from CroCo — method stub exists but no paper page). Deferred until a future ingest relies on CroCo's specific mechanism. InfoNCE, VCRE, Mean Average Accuracy — concepts only, not load-bearing enough for standalone pages at this ingest.

- `raw/` now empty.

## [2026-04-21] ingest | Feed-Forward 3D Scene Modeling: A Problem-Driven Perspective (Wang et al. 2026)

Second feed-forward-3D survey (66 pp., arXiv 2604.14025, CC-BY-NC-SA-4.0) ingested as **scaffolding, not as a mechanism contribution** — per-user guidance and Zhang 2025 precedent. No idea pages produced. The survey's distinctive contribution is its taxonomy: five problem-driven axes (feature enhancement, geometry awareness, model efficiency, augmentation, temporal-awareness) explicitly orthogonal to the output-representation axis covered by [[zhang2025_feed-forward-3d-survey]]. This cross-cutting vocabulary is captured as a first-class concept page.

- **Created:**
  - [wiki/papers/wang2026_feed-forward-3d-scene-modeling.md](wiki/papers/wang2026_feed-forward-3d-scene-modeling.md) — survey paper page; `sources: []` (survey is its own source); Pipeline contribution section lists thread-level scaffolding only, no idea-page creation.
  - [wiki/concepts/feed-forward-problem-axes.md](wiki/concepts/feed-forward-problem-axes.md) — **new concept page**. Captures the 5-axis taxonomy as cross-cutting vocabulary over existing wiki ideas; maps each axis to its primary thread home and representative in-wiki ideas. States (but does not enforce) a future `problem_axis:` frontmatter field — deferred until a second taxonomic source confirms the partition per CLAUDE.md §8.

- **Updated:**
  - [wiki/threads/feed-forward-structure-from-motion.md](wiki/threads/feed-forward-structure-from-motion.md) · op:default: Wang 2026 added to `sources:` and cross-cutting evidence section (alongside Zhang 2025). **Two new capability gaps added**: (1) view-selection-bias in standard benchmarks (RealEstate10K / ACID), distinct from the prior "outdoor generalization" open question which was about domain shift rather than protocol design; (2) VGGT-efficiency sub-family (FastVGGT / QuantVGGT / SparseVGGT) as a parallel direction to the TTT scaling story, with no in-wiki coverage and no head-to-head comparison to the TTT family on a common long-sequence benchmark. **Two new open questions added**: reconstruction-vs-generation spectrum (§7.6; shared with [[generative-3d-from-2d-priors]]) and gauge-decoupled streaming mechanism from LongStream (§4.5.1).
  - [wiki/threads/radiance-field-evolution.md](wiki/threads/radiance-field-evolution.md) · all OPs: Wang 2026 added to `sources:`. Three new open questions: the 4D-Gaussian feed-forward cluster (4DGT / L4GM / StreamSplat / DGS-LRM — candidate for a future `op:dynamic-4d` OP if the ≤3 cap allows); Gaussian-side compaction family (GGN / PixelGaussian / FreeSplat++ / LongSplat) as a complementary efficiency pathway to EA-3DGS's codebook VQ; video-world vs 3D-world model paradigm choice (§7.4).
  - [wiki/threads/generative-3d-from-2d-priors.md](wiki/threads/generative-3d-from-2d-priors.md) · op:default: Wang 2026 added to `sources:`. Existing "Reconstruction-vs-generation disentanglement" capability gap reinforced with §7.6's spectrum framing (continuum, not dichotomy). **New capability gap added**: video-world vs 3D-world paradigm choice (§7.4) — thread sits in the 3D-world paradigm; video-world variant not yet covered. "Bridge to multi-view reconstruction" gap linked to §4.4.2 Visual Augmentation as the mirror-image (reconstruction-first, generation-second) architecture to Bet #023.
  - [index.md](index.md) — added Wang 2026 paper entry under Fundamentals; added [[feed-forward-problem-axes]] under Concepts; bumped `_updated` dates on the three touched threads; rebuilt footer counts.

- **Pipeline impact:**
  - No SOTA-pipeline node changes on any thread — survey has no component-level contribution.
  - No new synthesis bets filed — survey literature is already in the wiki; no novel cross-paper combinations surface from a pure taxonomic source.
  - Pass B completed on all three touched threads with explicit "no new bets; taxonomy validated existing thread hypotheses, added 2 new capability gaps + 3 new open questions + 1 new concept page" outcome. This is the honest Pass B result for a survey.

- **Notable / non-obvious:**
  - **Why a concept page, not just thread notes.** Wang 2026's problem-axis taxonomy is strictly closer to the wiki's stage-filler idea schema than Zhang 2025's representation-axis. It is not redundant with the existing taxonomic scaffolding — it's a cross-thread *query vocabulary*. The concept page makes it addressable by wikilink; embedding it only in prose on three threads would have fragmented it.
  - **Orthogonality claim is asserted, not tested.** The concept page articulates how to operationalize the claim as a 5×5 problem × representation co-occurrence audit over the wiki's idea pages. Not filed as a bet because the audit is meta-lint-shaped rather than research-bet-shaped.
  - **Reconstruction-vs-generation spectrum is now a cross-thread primitive.** The [[generative-3d-from-2d-priors]] thread already carried this as a capability gap from the SAM 3D ingest; Wang 2026 §7.6 now promotes it to an open question on [[feed-forward-structure-from-motion]] as well. The two threads share a seam.
  - **License shift.** Wang 2026 is `CC-BY-NC-SA-4.0` (non-commercial) — second non-commercial paper in the wiki after MASt3R. Immaterial for the wiki's current purposes (no reproduction of the paper text); flagged in the paper page's Code & license section for completeness.
  - **Search targets for future ingest** (flagged but not pursued per Minimal-Medium-Full scope choice): LongStream (gauge-decoupled streaming visual geometry), Stream3R (decoder-only causal Transformer for pointmaps), VGGT-efficiency cluster (FastVGGT / QuantVGGT / SparseVGGT), 4D-Gaussian cluster (4DGT / L4GM / StreamSplat / DGS-LRM), MegaSynth (procedural non-semantic augmentation).

- **Not created (deferred per §6.5 "name-drop is not a page"):** 30+ methods cited once in Wang 2026 (iLRM, Long-LRM, ZPressor, TinySplat, NoPoSplat, Splatt3R, PF3plat, FreeSplatter, FLARE, RegGS, GGN, PixelGaussian, FreeSplat++, LongSplat, Puzzles, Aug3D, Easi3R, 4D-LRM, L4GM, 4DGT, StreamSplat, DGS-LRM, Stream3R, LongStream, MonST3R, PIXIE, PhysGM, ARTDECO, EC3R-SLAM, MASt3R-Fusion, ViSTA-SLAM, SLAM3R, VGGT-SLAM, VGGSfM, Light3R-SfM). None are referenced ≥2 times across the wiki or materially advance any current thread beyond being listed.

- `raw/` now empty.

## [2026-04-21] ingest | VGGT-efficiency cluster (Shen 2025 FastVGGT + Wang 2025 Faster-VGGT block-sparse + Feng 2025 QuantVGGT)

Three-paper batch closing the "VGGT-efficiency sub-family" capability gap that the Wang 2026 survey ingest had flagged as a Search target earlier today. All three papers target the same bottleneck — quadratic global-attention cost in VGGT/π³ — from three genuinely orthogonal resource axes: **token count** (FastVGGT), **attention sparsity** (Faster-VGGT block-sparse, which Wang 2026 informally named "SparseVGGT"), and **numerical precision** (QuantVGGT W4A4 PTQ). None of the three papers tests composition with the others; that untested composition is the natural cross-paper synthesis (Bet #029) and the wiki's reason for existing.

- **Created:**
  - [wiki/papers/shen2025_fastvggt.md](wiki/papers/shen2025_fastvggt.md) · arXiv 2509.02560 · ICLR 2026 · Meta VGGT License (research-only, non-commercial).
  - [wiki/papers/wang2025_faster-vggt-block-sparse.md](wiki/papers/wang2025_faster-vggt-block-sparse.md) · arXiv 2509.07120 · no code license declared.
  - [wiki/papers/feng2025_quantvggt.md](wiki/papers/feng2025_quantvggt.md) · arXiv 2509.21302 · ICLR 2026 · CC-BY-4.0 paper + Apache-2.0 code — the **only commercial-clean** paper of the three.
  - [wiki/ideas/vggt-token-merge-3-part-partition_shen2025.md](wiki/ideas/vggt-token-merge-3-part-partition_shen2025.md) · scope `stage-split` (introduces new pre-attention stage).
  - [wiki/ideas/pooled-qk-block-sparse-global-attention_wang2025.md](wiki/ideas/pooled-qk-block-sparse-global-attention_wang2025.md) · scope `stage-swap` on existing `feed-forward-sfm.global-attention`.
  - [wiki/ideas/vggt-dual-smoothed-quantization_feng2025.md](wiki/ideas/vggt-dual-smoothed-quantization_feng2025.md) · scope `stage-swap` on new `feed-forward-sfm.numerical-precision`.
  - [wiki/ideas/frame-aware-ptq-calibration-sampling_feng2025.md](wiki/ideas/frame-aware-ptq-calibration-sampling_feng2025.md) · scope `drop-in` on new `feed-forward-sfm.ptq-calibration-sampling`, **`co_requires: vggt-dual-smoothed-quantization_feng2025`** (bundle).
  - [wiki/stages/feed-forward-sfm.token-compaction.md](wiki/stages/feed-forward-sfm.token-compaction.md) — new pre-attention stage for token-count reduction in VGGT-family pipelines.
  - [wiki/stages/feed-forward-sfm.numerical-precision.md](wiki/stages/feed-forward-sfm.numerical-precision.md) — new post-training-transform stage for PTQ/quantization.
  - [wiki/stages/feed-forward-sfm.ptq-calibration-sampling.md](wiki/stages/feed-forward-sfm.ptq-calibration-sampling.md) — new calibration-sample-selection stage (companion to numerical-precision).

- **Updated:**
  - [wiki/threads/feed-forward-structure-from-motion.md](wiki/threads/feed-forward-structure-from-motion.md) · op:default (Tier 3):
    - Added three papers to `sources:`.
    - Added three Tier-3 evidence entries.
    - **Closed the "VGGT-efficiency sub-family" capability gap** (partially): the *existence* gap is now closed (four ideas land in the wiki); the *comparison* gap vs the TTT-scaling family remains open as Bet #030's motivation.
    - Added pattern observation to "Emerging patterns": VGGT-compression family as parallel scaling trick alongside TTT-scaling family, with the key mechanistic distinction — compression preserves the attention function class, TTT replaces it.
    - **Filed two new Pass B synthesis bets**:
      - **Bet #029 — Triple-orthogonal VGGT compression stack** (token-merge × block-sparse × W4A4): composition of all three VGGT-compression ideas plus QuantVGGT's bundle. Expected ~40× end-to-end speedup, ~14× memory reduction at ≤2% accuracy loss. None of the 3 papers tested composition — this is a genuine cross-paper novel combination.
      - **Bet #030 — VGGT-compression vs TTT-scaling family head-to-head**: direct Pareto comparison on a common long-sequence benchmark, addressing the comparison gap. First publication-quality comparison of the two Tier-3 scaling branches.
  - [index.md](index.md) — 3 new paper entries under SfM / SLAM; stage count bumped from 93 → 96 with new stages enumerated; idea count bumped 87 → 91; footer rebuilt.

- **Pipeline impact:**
  - No SOTA-pipeline node *changes* on any op (these are efficiency transforms, not accuracy-improving SOTA swaps). Instead, they populate a parallel branch of candidate fillers at the long-context-attention stage, orthogonal to the TTT-family fillers currently there.
  - Pass B completed with **two concrete bets** — closing the loop on the Wang 2026 ingest's capability-gap-refresh in a single day. This is the end-to-end synthesis cycle the wiki is designed to produce: survey flags a gap → search targets identified → papers ingested → bets filed proposing novel compositions.

- **Notable / non-obvious:**
  - **Orthogonality as a synthesis lever.** The three papers are the clearest case in the wiki so far of mechanisms attacking *genuinely independent resource axes* on a shared bottleneck. Each paper only ablates its own axis. The triple-composition Bet #029 is therefore a pure "cross-paper bet" — no single paper's authors could have conceived it; it only becomes visible once all three are in the same wiki.
  - **License diversity within a family.** FastVGGT inherits Meta's VGGT License (non-commercial research-only, via its LICENSE.txt). Faster-VGGT has *no* code license declared (all-rights-reserved by default on GitHub). QuantVGGT is CC-BY-4.0 paper + Apache-2.0 code — commercial-clean. The License Audit meta page should be updated to reflect this spread; for Bet-level planning, only QuantVGGT's axis is commercial-adoptable without legal review. Per CLAUDE.md §6.15, bet composition decisions are unaffected — but downstream implementation readiness is materially different across the three.
  - **FastVGGT's drift-mitigation finding.** Long-sequence pose *improves* (not just matches) under 90% token merging. The paper frames it as a cumulative-drift mitigation but does not isolate the mechanism experimentally — intriguing open question for future interpretability work on VGGT's aggregator behavior.
  - **NFDS as a transferable contribution.** QuantVGGT's frame-aware calibration sampling is the contribution most likely to transfer beyond VGGT — deep-layer noise filtering is architecture-agnostic and the frame-distribution-clustering premise extends to any multi-view-structured input. Potential cross-thread transfer target for future PTQ work on dense matchers / monocular depth foundation models.
  - **Stage design choices.** Chose three new stages over collapsing into one `feed-forward-sfm.backbone-efficiency` composite, because FastVGGT's token compaction is compositional with *any* downstream attention-mechanism filler (not just its own), SparseVGGT replaces the attention itself (same stage), and QuantVGGT operates at a deployment-time transform layer orthogonal to attention. Three stages, three distinct composition semantics. Matches CLAUDE.md §1.7's discipline: "Stage IDs are dotted: `<domain>.<slot>`."
  - **Scope classification was the judgment call.** FastVGGT initially presented as `stage-swap` in the Step 2 plan but revised to `stage-split` on write — it genuinely introduces a new pre-attention stage, not just a different filler for an existing one. Classifying as stage-swap would have flattened the fact that the token-compaction node is independent of the attention-mechanism node and can receive *any* downstream filler.

- **Not created (deferred):** π³ (Wang 2025c, permutation-invariant VGGT variant) is referenced by all three papers but no wiki page yet — would be a reasonable near-term stub if ingest load grows. SpargeAttention (LLM-world sparse attention precedent cited by Faster-VGGT) is outside the wiki's photogrammetry/ML scope. QuaRot, SpinQuant, SmoothQuant (PTQ baselines) — rotation-based PTQ lineage; not load-bearing for wiki threads.

- `raw/` now empty.

## [2026-04-21] ingest | Lyra 2.0: Explorable Generative 3D Worlds (Shen et al. 2026)

Single-paper ingest from `raw/2604.13036v1.pdf` (13 MB arxiv-named PDF). Lyra 2.0 is the **canonical video-world representative** the wiki lacked — directly closes the "video-world vs 3D-world paradigm choice" capability gap that the prior Wang 2026 ingest flagged on both [[generative-3d-from-2d-priors]] and [[radiance-field-evolution]]. 15 NVIDIA Toronto / SIL authors. Apr 2026.

- **local_paper:** papers/radiance-fields/shen_2026_lyra-2-explorable-3d-worlds.pdf. Code: [github.com/nv-tlabs/lyra](https://github.com/nv-tlabs/lyra) **Apache-2.0** (commercial-friendly — unusual for a Wan-2.1-14B-based system; upstream weight licenses may compose). Weights: [huggingface.co/nvidia/Lyra-2.0](https://huggingface.co/nvidia/Lyra-2.0) (license not independently audited; flagged for `lint licenses`). Paper: arxiv-nonexclusive.

- **Created paper:** [wiki/papers/shen2026_lyra2.md](wiki/papers/shen2026_lyra2.md).

- **Created ideas (6 atomic, 2 bundles, 1 unbundled cross-thread candidate):**
  - [[per-frame-3d-cache-retrieval_shen2026]] · `scope: stage-swap` at [[video-world.spatial-memory]] (new stage). Per-frame 3D cache (full-res depth + downsampled point cloud at `d=8`) + visibility-score greedy-coverage retrieval (`N_s=5`). Per-frame independence — no global fusion. **Bundle: `co_requires` idea 2.** Load-bearing ablation: Table 3 "w/ Global Point Cloud" row → **Camera Controllability 63.87 → 49.86 (−14.01)** on Tanks-and-Temples — the largest single ablation swing in the paper.
  - [[canonical-coord-warp-injection_shen2026]] · `scope: stage-swap` at [[video-world.correspondence-injection]] (new stage). Forward-warped 4-channel `[Ĉ_j; D̂_j]` canonical coordinate maps → MLP+PE aggregator → inject on Q/K only of DiT self-attention (zero-init). Non-obvious sub-mechanism: Q/K-only no-V injection preserves pretrained value projection while additively learning correspondence guidance. **Bundle partner of idea 1.** Partial isolation: Table 3 "w/ Explicit Corr. Fusion" → −6.58 CC (tests aggregation mechanism but not canonical-coord-vs-warped-RGB choice; flagged on idea page).
  - [[self-augmented-history-training_shen2026]] · `scope: drop-in` at [[video-world.drift-mitigation-training]] (new stage). One-step flow-matching noise injection on clean-latent history with `p_aug=0.7`, `t ~ U(0, 0.5)`. ~35× cheaper than Self-Forcing on bi-directional DiTs. Isolated ablation: Table 3 "w/o Self-Augmentation" → **Style Consistency −7.09, Camera Controllability −9.95**. Exposes an **honest trade-off**: per-frame Subjective Quality *improves* without aug (+4.53) — long-range consistency traded for short-range polish. **Cross-thread transfer candidate** — architecture-agnostic, applies to any autoregressive flow/diffusion model.
  - [[downsampled-gaussian-dpt-head_shen2026]] · `scope: drop-in` at [[generative-3d.shape-generation]] (existing). `k × k` strided Gaussian DPT head on DAv3 (k=2, 4× fewer Gaussians). **Evidence weaker — no isolated ablation**; flagged on idea page. Paired with a fine-tune on generated data per Lyra 1 precedent (not novel; not a separate idea page).
  - [[dmd-with-self-aug_shen2026]] · `scope: drop-in` at [[video-world.distillation]] (new stage). DMD 35→4 step + CFG distilled + **self-augmentation retained during distillation** so student inherits drift robustness. ~13× per-step speedup (194s → 15s per 80-frame step on GB200). Student remains SOTA above all non-Lyra baselines. **Bundle: `co_requires` idea 3** (retention of a property the teacher doesn't have is vacuous).
  - [[hierarchical-sparse-grid-mesh-extraction_shen2026]] · `scope: stage-swap` at [[mesh-reconstruction.extraction]] (existing). OpenVDB adaptive-level sparse grid + SDF from 3DGS depth + per-level marching cubes + level-transition merge. **Evidence weaker — no isolated ablation**; qualitative-only via Isaac Sim demo; flagged on idea page.

- **Created stages (4, new `video-world.*` prefix):**
  - [[video-world.spatial-memory]] — which history frames to recall when target is outside temporal context window.
  - [[video-world.correspondence-injection]] — how to convey geometric correspondence to the backbone without leaking appearance.
  - [[video-world.drift-mitigation-training]] — training-time protocol to close the observation-bias gap.
  - [[video-world.distillation]] — deployment-time step reduction that must preserve teacher-side drift properties.

- **Created concept:** [[information-routing-vs-3d-rendering-memory]] — the design-principle dichotomy Lyra 2.0 organizes its anti-forgetting argument around (3D-as-routing vs 3D-as-rendering-memory). Referenced by Ideas 1 + 2 and by the concept's own taxonomy of existing video-world methods (Gen3C, SPMem, VMem, Context-as-Memory, WorldMem — currently all not-in-wiki).

- **Created stubs:**
  - [[bahmani2025_lyra]] — Lyra 1 paper stub (arXiv 2509.19296, Sep 2025). Direct precedent to Lyra 2.0; establishes the generative-reconstruction paradigm + the fine-tune-reconstructor-on-generated-data recipe Lyra 2.0 inherits.
  - [[depthanythingv3]] — DAv3 method stub. Load-bearing as Lyra 2.0's depth backbone AND as the feed-forward 3DGS reconstructor (distinct from DAv1/v2 which this wiki already has as [[depthanything]]). `> [!needs-source]` on both.

- **Updated:**
  - [[generative-3d-from-2d-priors]] · **new OP `op:explorable-scene`** (the thread now has 2 OPs — under the ≤3 cap; `op:default` for SAM-3D-style object generation, `op:explorable-scene` for Lyra-2-style video-world scene generation). **New SOTA DAG**: 7 nodes (n4-n10) covering spatial memory → correspondence injection → drift-mitigation training → DiT backbone → feed-forward 3DGS head → mesh extraction → distillation. Topology-change lineage entry filed. **Three new bets** (#024–#026): self-aug as causal-backbone Self-Forcing replacement (high-confidence cross-thread cost-reduction); Lyra-video-world output as multi-view prior for op:default SAM-3D (paradigm-bridging, cross-OP); canonical-coord injection recipe transferred to SAM 3D's shape generator (incremental, Q/K-only recipe transfer). **Capability-gaps refreshed**: video-world-vs-3D-world gap closed; new gaps filed for rendering-memory variants not in wiki, long-horizon (2K+ frame) evaluation protocol, dynamic-scene generalization, photometric stability. Four new `sources:` added.
  - [[gaussian-to-mesh-pipelines]] · [[hierarchical-sparse-grid-mesh-extraction_shen2026]] added to **Candidate components** (did not win any OP — no isolated ablation in source paper, and current OP benchmarks are object/room scale where hierarchical allocation isn't load-bearing). New **capability gap**: large-scale scene extraction benchmark — no in-wiki filler has been quantitatively demonstrated at scene scale; would motivate `op:large-scale-scene` if a head-to-head benchmark existed. Sources updated.
  - [[radiance-field-evolution]] · [[downsampled-gaussian-dpt-head_shen2026]] added to **Candidate components** (target OPs `op:city-scale` and `op:quality-per-scene`; did not displace current fillers because current OPs use per-scene optimization rather than feed-forward Gaussian prediction — no directly-comparable target). **Two open questions refreshed**: video-world-vs-3D-world paradigm (partial answer: paradigms are not zero-sum, radiance field can live downstream of video-world generator); feed-forward generative-reconstruction as alternative to per-scene optimization (insufficient evidence for an OP split yet). Sources updated.
  - [[feed-forward-structure-from-motion]] · **Capability gap expanded** on "reconstruction-vs-generation spectrum" — Lyra 2.0 is now the canonical generation-first-reconstruction-second in-wiki exemplar; [[per-frame-3d-cache-retrieval_shen2026]] is flagged as a potential cross-thread import for visibility-based long-context frame selection (orthogonal to TTT-family compute scaling). Sources updated (Lyra 2.0 added for the cross-thread capability-gap reference).
  - [index.md](index.md) — added Lyra 1 stub + Lyra 2.0 under Radiance Fields papers; added DAv3 under Methods; added information-routing concept; bumped `updated:` on touched threads; refreshed idea/stage counts (91→97 ideas, 96→100 stages) + footer tallies.

- **Pipeline impact:**
  - New OP on [[generative-3d-from-2d-priors]] instantiated end-to-end with 7-node DAG under `op:explorable-scene`. Topology-change lineage entry: `scope: new-paradigm`. The existing `op:default` is preserved unchanged.
  - No SOTA-pipeline node changes on [[radiance-field-evolution]] or [[gaussian-to-mesh-pipelines]] — Lyra 2.0 ideas land in Candidate components on those threads.
  - 3 new synthesis bets filed (#024, #025, #026 on the primary thread). Two bets (#024 cross-thread self-aug adoption; #025 cross-OP video-world-as-sparse-view-prior) are the Pass B novel-combination payoff. Pass B explicitly enumerated over [[per-frame-3d-cache-retrieval_shen2026]]'s `unlocks:` + [[self-augmented-history-training_shen2026]]'s cross-thread transferability + the two OPs' now-proximal composition.

- **Notable / non-obvious:**
  - **Q/K-only no-V zero-init injection** is Lyra 2.0's most transferable non-headline trick. Captured on [[canonical-coord-warp-injection_shen2026]] Open questions — likely to appear in Pass B bets beyond the original video-world context (any "bolt guidance onto a pretrained foundation model" scenario).
  - **Self-augmentation vs Self-Forcing cost gap (~35×)** is the cheap-path enabler for applying drift-mitigation training to large bi-directional video DiTs. Bet #024 is the direct cross-thread adoption play. This is also the idea most likely to recur in future Pass B rounds — architecture-agnostic, training-only, no inference graph changes.
  - **Information-routing-vs-rendering concept** makes a taxonomic cut that neither Zhang 2025 (output-representation axis) nor Wang 2026 (problem-driven axis) capture — it is a design-principle axis *inside* the temporal-awareness family. [[feed-forward-problem-axes]] is orthogonal to it. The ability to do this as a graph query ("which ideas commit to routing?") is exactly why the concept page exists rather than being prose on three threads.
  - **Two OPs same thread, not new thread**: kept `op:explorable-scene` on [[generative-3d-from-2d-priors]] rather than forking because the input regime is identical (single image → 3D scene); only the intermediate representation (3D latent vs video-lifted 3D) differs. This is the right call only if the two OPs genuinely compete for the same task objective — they do (user wants a 3D scene from a photo). Bet #025 is the cross-paradigm composition that makes this co-location load-bearing rather than arbitrary.
  - **Trade-off surfaced in ablation row**: w/o Self-Augmentation actually *improves* per-frame Subjective Quality (+4.53). The paper does not hide this. Captured on [[self-augmented-history-training_shen2026]] — for short-video deployments, disabling augmentation may be correct. This is the kind of user-facing trade-off that the wiki's ideas should surface rather than bury.
  - **Apache-2.0 code license is unusual** in this research cluster. [[leroy2024_mast3r|MASt3R]] (CC-BY-NC-SA), Lyra 2.0 Wan-2.1 backbone (upstream license likely restrictive) compose differently for commercial deployment. License audit at [[license-audit]] should register Apache-2.0 for Lyra 2.0 code + note effective-license-depends-on-Wan-weights.
  - **Evidence-quality stratification**: Ideas 1 + 3 + 5 have strong/partial isolated ablations; Ideas 2 + 4 + 6 are flagged as weaker (no isolated row / qualitative-only). Every idea page carries this on its Open questions section. Pass B bets weighted accordingly (Bet #024 on Idea 3 = high-confidence; hypothetical bets on Idea 6 = lower-confidence if proposed).

- **Not created (deferred per §6.5 "name-drop is not a page"):** Wan 2.1 [107] (video backbone — load-bearing only for this paper; no stub yet), FramePack [135] (used as-is), Self-Forcing [36] (cited as contrast, worth ingesting if a second mentioning lands), DMD [126] (used as-is), DL3DV [60] (training dataset — single-paper reference; if a second paper uses DL3DV evaluation, consider dataset page), ViPE [35] (pose estimator used for training data curation — single-paper reference), Qwen3-VL-8B-Instruct [103] (caption model — outside scope), OpenVDB [72, 115] (implementation library — outside scope).

- **Search targets for future ingest** (flagged but not pursued):
  - **Gen3C** [81] — strongest global-3D-memory baseline per Lyra 2.0 Table 1 (70.91 CC on T&T); would let the wiki quantitatively confirm the routing-vs-rendering-memory distinction against an in-wiki baseline.
  - **SPMem** [117] — strongest CaM-family multi-view memory baseline; mentioned-but-not-public (Lyra 2.0 re-implements).
  - **Self-Forcing** [36] — causal-architecture drift mitigation; Bet #024's trigger paper.
  - **WorldMem** [118], **Context-as-Memory** [128], **VMem** [51] — retrieval-based video-world memory family that would populate [[information-routing-vs-3d-rendering-memory]]'s examples with in-wiki references.
  - **Depth Anything v3** (Yang 2026 [58]) — load-bearing at two Lyra 2.0 stages; [[depthanythingv3]] is currently a `> [!needs-source]` stub.
  - **Lyra 1** (Bahmani 2025 [2], arXiv 2509.19296) — direct precedent; [[bahmani2025_lyra]] stub is in wiki.

- `raw/` now empty.
