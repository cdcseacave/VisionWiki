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
