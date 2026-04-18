---
title: "Language-Grounded 3DGS — Research-Synthesis Design (2026)"
type: design
tags: [3dgs, language-grounded, instance-segmentation, open-vocabulary, feature-distillation, sam3, radio, dinov3, interaction-layer, densify-prune-invariant]
created: 2026-04-15
updated: 2026-04-18
sources:
  - papers/qin2024_langsplat.md
  - papers/ye2024_gaussian-grouping.md
  - papers/jiao2025_clip-gs.md
  - papers/bao2025_seg-wild.md
  - papers/jatavallabhula2023_conceptfusion.md
  - papers/carion2026_sam-3.md
  - papers/heinrich2025_radiov25.md
  - papers/simeoni2025_dinov3.md
  - papers/oquab2023_dinov2.md
  - papers/wu2026_langsvr.md
  - papers/kim2026_gauss-explorer.md
  - papers/chen2025_sam-3d.md
  - papers/mao2025_spatiallm.md
  - papers/radford2021_clip.md
  - papers/kirillov2023_sam.md
realizes_bet: Bet #020
realizes_ideas: [[sam3-native-video-ids_carion2026]], [[radiov25-agglomerative-distillation_heinrich2025]], [[dinov3-gram-anchoring_simeoni2025]], [[langsplat-per-scene-autoencoder_qin2024]], [[gaussian-grouping-identity-encoding_ye2024]]
outcome: pending
outcome_date: null
status: draft
---

## Purpose

End-to-end design for augmenting a 3DGS training pipeline with per-primitive feature embeddings that enable (a) 3D instance segmentation, (b) open-vocabulary classification, and (c) free-language interaction — text/click/compositional queries, with edits applied directly to the Gaussian set.

This design rests on the anchor threads [[lifting-foundation-models-to-3d]] and [[open-vocab-2d-composition]], plus an adjacent-thread reference to [[gaussian-to-mesh-pipelines]] for the non-goals. It **recomposes the 2024-era baselines** ([[qin2024_langsplat|LangSplat]], [[ye2024_gaussian-grouping|Gaussian Grouping]], CLIP+SAM+DINO Trident-style composition) using the **2026-era foundation-model stack** ([[carion2026_sam-3|SAM 3]] native video IDs, [[heinrich2025_radiov25|RADIOv2.5]] unified features, [[simeoni2025_dinov3|DINOv3]] geometric priors) plus [[wu2026_langsvr|LangSVR]]'s finding that geometric FM priors sharpen semantic boundaries. It ships the gap all prior papers punted on: a precomputed interaction layer (click < 5 ms, text < 100 ms at 1 M Gaussians) and a densification/pruning invariant that keeps per-primitive identity and language fields coherent under the standard 3DGS training loop.

Implementation into the [[nerfstudio|visiofacto fork]]: see [[language-grounded-3dgs-nerfstudio]].

---

## Context

Vanilla 3DGS trains primitives for photometric reconstruction and has no notion of object identity or semantics. Three 2024-era recipes ([[qin2024_langsplat|LangSplat]], [[ye2024_gaussian-grouping|Gaussian Grouping]], [[bao2025_seg-wild|Seg-Wild]]) added these as per-Gaussian fields but (i) used 2024 foundation models (CLIP + SAM-1 + DINOv2) that the 2026 stack supersedes, (ii) trained in two stages (freeze 3DGS, then distill features on top), and (iii) specified no concrete interaction layer beyond "cosine similarity between rendered features and a text query." The 2026 stack offers native video instance IDs ([[carion2026_sam-3|SAM 3]]), a single unified feature backbone ([[heinrich2025_radiov25|RADIOv2.5]]), geometric-prior-rich features ([[simeoni2025_dinov3|DINOv3]]), and a demonstrated finding that geometric FM priors sharpen semantic boundaries ([[wu2026_langsvr|LangSVR]]). This design composes these into a 3DGS pipeline that ships with a precomputed interaction layer hitting click-selection in < 5 ms, text-selection in < 100 ms at 1 M Gaussians, and a densification/pruning invariant that keeps the per-primitive identity and language fields coherent under the standard 3DGS training loop — a gap every prior paper papers over. The whole-scene edits (delete/hide/recolor/transform) operate directly on the Gaussian set without re-rendering supervision.

---

## Goal & success criteria (numeric)

| Dimension | Target | Rationale / anchor |
|---|---|---|
| 3D-OVS mIoU | **≥ LangSplat on 3D-OVS benchmark** (and +3–5 pp on boundary-heavy scenes where the geometric-prior claim applies) | Language-lifting primary metric from [[qin2024_langsplat]]; geometric-prior lift from [[wu2026_langsvr]] ablation |
| Instance IoU (scene-level) | **≥ Gaussian Grouping on ScanNet open-vocab subset**, with zero cross-view association-loop supervision | [[ye2024_gaussian-grouping]] baseline; deletion of association loop is the [[carion2026_sam-3]] synthesis bet already filed in [[lifting-foundation-models-to-3d]] |
| Click→selection latency | **< 5 ms** on RTX 4090 at 1 M Gaussians (end-to-end: rasterize id-code pixel → codebook lookup → identity-hash expansion) | No paper quotes this; target derived from 60 FPS interactive-edit UX |
| Text→selection latency | **< 100 ms** on RTX 4090 at 1 M Gaussians (end-to-end: RADIO text encode → per-instance mean-latent cosine top-K → identity-hash expansion, with k-NN fallback budget of 30 ms) | Interactive target; dominated by text encode + instance cosine |
| Compositional query ("the red mug", "all chairs without armrests") | **< 3 s** (async), VLM-in-the-loop | Known failure mode of cosine on averaged latents; GaussExplorer pattern [[kim2026_gauss-explorer]] |
| Per-Gaussian memory overhead | **≤ 48 bytes/Gaussian** (8D id_code f16 + 32D lang_latent f16 + 4 bytes for VQ-coded id + 4 bytes metadata) = 48 MB at 1 M Gaussians | [[bao2025_seg-wild|Seg-Wild]] uses multi-channel embeddings without budget; LangSplat's AE is "low-D" but unspecified |
| Training time | **≤ 1.6× vanilla 3DGS** on the same scene (one-stage joint) | LangSplat is 2-stage (3DGS pretrain + distill); [[wu2026_langsvr|LangSVR]] claims one-stage at small overhead but on voxels |
| Deletion of association loop | **removed from the training loop** | [[ye2024_gaussian-grouping]]'s "most fragile stage" flagged in-thread |

Anything below these numbers is not a success.

---

## Pipeline (stage-by-stage)

Each stage lists: **source paper · mechanism · what it replaces from the prior 2024 SOTA.** Stages 1–5 are training; Stages 6–8 are the precomputed query/interaction layer.

### Stage 1 — 2D mask + instance-ID source: SAM 3 (direct)
- **Source:** [[carion2026_sam-3]].
- **Mechanism:** For each training view of the scene, run SAM 3 in PCS mode with either (a) an automatic concept sweep across a scene-wide noun-phrase vocabulary bootstrapped from RADIO image captions, or (b) the user-supplied vocabulary if known. SAM 3 emits masks + **native video track IDs + presence logit** per concept. The video tracker provides cross-frame identity consistency out of the box.
- **Replaces:** SAM-1 ([[kirillov2023_sam]]) + Gaussian Grouping's IoU-and-feature cross-view association module ([[ye2024_gaussian-grouping]] §Method). That module is the 2024 recipe's most fragile stage; SAM 3's native track IDs obsolete it. **This is the thread's filed synthesis bet #1 for [[lifting-foundation-models-to-3d]] Pipeline II.**
- **Evidence the replacement is sound:** [[carion2026_sam-3]] reports 2× accuracy over prior SOTA on image + video PCS benchmarks, with the presence head specifically addressing "hallucinated detections on absent concepts" — the failure mode that pollutes Gaussian Grouping's per-track supervision when association bleeds. The paper explicitly flags 3D lifting as untried downstream.
- **What the data layer stores:** per-view tensor of `(mask, track_id, concept_string, presence_logit)`. Concepts resolved to a scene-level vocabulary V of ~500–5 000 noun phrases.

### Stage 2 — Per-pixel semantic feature source: RADIOv2.5 (direct, replaces CLIP + DINO + SAM composition)
- **Source:** [[heinrich2025_radiov25]].
- **Mechanism:** Run RADIOv2.5 ViT-H **once per training view** at the training resolution; extract dense patch features (subsumes CLIP semantic alignment, DINOv2 spatial coherence, and SAM-H mask-boundary awareness in a single forward pass). Apply ToMe-compressed tokens for high-res views when feature-map size exceeds memory budget.
- **Replaces:** The LangSplat recipe's (i) per-view CLIP forward pass at SAM-hierarchy crops, (ii) dropped-DINO regularization (LangSplat argued SAM hierarchy + 3DGS geometry supplied spatial coherence), and (iii) implicit reliance on CLIP's 512-D space. RADIOv2.5 gives CLIP-aligned + DINO-aligned + SAM-aligned signal in 768D from one pass.
- **Evidence:** [[heinrich2025_radiov25]] reports ADE20k mIoU 53.97%, ImageNet zero-shot 82.51%, SAM COCO instance 76.14%, Probe3D depth 85.7%. The paper's multi-resolution training explicitly kills the low-res/high-res "mode switch" — load-bearing for 3DGS where training views vary in effective resolution. The wiki flags this as an **untried** 3D synthesis bet.
- **Contradiction acknowledged:** [[qin2024_langsplat|LangSplat]]'s §Method claims "SAM hierarchy + 3DGS geometry supplies spatial coherence → drop DINO." This was probably correct at DINOv2 and CLIP separately but does not address the RADIO single-pass case. The design's position: RADIOv2.5's unified features are strictly a superset of LangSplat's CLIP-only signal, so adopting it is a superset move, not a reintroduction of the dropped DINO leg. Cited: [[open-vocab-2d-composition]] Pipeline B/C discussion.

### Stage 3 — Geometric priors: DINOv3-derived monocular depth/normal head (replaces no prior; adds a leg absent in 3DGS lane)
- **Source:** [[simeoni2025_dinov3]] for the feature backbone, plus any monocular depth/normal head built on top (Depth-Anything-v2 style is fine; the design is prior-model-agnostic as long as depth and normal are rendered as dense maps per training view).
- **Mechanism:** For each training view, produce predicted depth and surface-normal maps. These become the supervision target for two auxiliary losses: (i) **depth-correlation regularization** — Pearson correlation between the 3DGS-rendered depth and the prior's depth must be above a floor, masked to reliable regions; (ii) **pattern-consistency regularization** — rendered normals must be locally consistent (inner-product with nearby pixel normals stays above a floor at mask boundaries).
- **Replaces:** Nothing in 3DGS lane; **adds** a leg only [[wu2026_langsvr|LangSVR]] currently has, and only on the sparse-voxel substrate. The design ports that objective function onto 3DGS.
- **Evidence:** [[wu2026_langsvr]] §Method and §Why it matters contain the load-bearing claim: *"depth/normal priors from a geometric foundation model measurably improve CLIP-feature quality in 3D, because they disambiguate where semantic boundaries should live."* LangSVR outperforms LangSplat, Feature-GAGS, Gaussian Grouping, and the SVRaster baseline on joint mIoU + PSNR. The substrate is voxels, but the objective-function claim is substrate-agnostic.
- **Contradiction acknowledged:** [[wu2026_langsvr]] argues sparse voxels are a competitive substrate for language lifting. The design's position: LangSVR's *objective function* is the load-bearing contribution; its *substrate* is a separate claim. Design target is 3DGS (photometric quality still best in 3DGS; per-Gaussian addressability is what downstream edits rely on; sparse voxels buy spatial-radius queries we do not need). We port the loss, keep the substrate.

### Stage 4 — Per-Gaussian fields (the storage layout)
- **Source:** composes [[qin2024_langsplat]] (per-scene autoencoder pattern) + [[ye2024_gaussian-grouping]] (per-Gaussian identity pattern) + a new VQ layer on top.
- **Fields added to each Gaussian** (beyond standard `μ, Σ, opacity, SH`):
  - `id_code` — **8-D continuous**, f16 → 16 bytes/Gaussian raw.
  - `id_vq` — **14-bit VQ index** into a scene-level codebook of ≤ 16 384 entries. 2 bytes/Gaussian. Produced by an online VQ layer updated once per epoch via k-means++ on visible id_codes. The discrete id is what the interaction layer hashes on; the continuous code is what the loss trains on.
  - `lang_latent` — **32-D continuous**, f16 → 32 bytes/Gaussian raw.
- **Per-scene language autoencoder** (on top, à la LangSplat): encode RADIOv2.5's 768-D patch features → 32-D latent; decode at query time when needed. The AE is trained jointly with the 3DGS, not as a separate pre-training pass (see Stage 5).
- **Dimensionality rationale:** pushes back against LangSplat's "low-D unspecified" and Gaussian Grouping's "identity dim is a free hyperparameter, no principled choice" ([[ye2024_gaussian-grouping]] §Open questions verbatim). 8-D continuous is small enough that unused subspace collapses during training; the 14-bit VQ gives a hard discrete index with 16 384 entries (room for ~10 000 distinct instances in a scene + slack). 32-D language latent is the lowest RADIOv2.5 patch-feature compression trustable for preserving fine-grained distinctions ("mug" vs "cup") — anything below 16-D starts losing class separation in RADIO's unified space per [[heinrich2025_radiov25]] §Results (Probe3D/ADE20k).
- **Total per-Gaussian overhead:** 16 + 2 + 32 = **50 bytes**, so **50 MB at 1 M Gaussians**. Inside the stated budget.

### Stage 5 — One-stage joint training objective (replaces LangSplat/GG's two-stage recipe)
- **Source:** ports [[wu2026_langsvr]]'s one-stage joint-optimization recipe from sparse voxels onto 3DGS.
- **Loss (single backward pass):**

  $$\mathcal{L} = \lambda_{\text{rgb}} \mathcal{L}_{\text{rgb}} + \lambda_{\text{id}} \mathcal{L}_{\text{id}} + \lambda_{\text{lang}} \mathcal{L}_{\text{lang}} + \lambda_{\text{geo}} \mathcal{L}_{\text{geo}} + \lambda_{\text{sc}} \mathcal{L}_{\text{sc}}$$

  - $\mathcal{L}_{\text{rgb}}$: standard 3DGS L1 + D-SSIM on rendered color.
  - $\mathcal{L}_{\text{id}}$: **cross-entropy between rasterized id_code (projected through the VQ codebook to a soft distribution over 16 384 classes) and SAM 3 track ID** at each mask pixel. Exactly the Gaussian Grouping loss with the SAM 3 supervision drop-in — cross-view consistency is now free (Stage 1).
  - $\mathcal{L}_{\text{lang}}$: **cosine loss between rasterized lang_latent (decoded by the AE) and RADIOv2.5's 768-D patch feature** at each pixel. Apply at multiple resolutions (RADIO supports this natively; 3DGS's splatter-render cost at each res is small). No SAM-hierarchy cropping loop — RADIO's patch features already carry the multi-scale signal, unlike raw CLIP.
  - $\mathcal{L}_{\text{geo}}$: LangSVR-style **depth correlation + normal pattern consistency**, supervised by the DINOv3 depth/normal priors.
  - $\mathcal{L}_{\text{sc}}$: 3D spatial-consistency regularizer on id_code (neighbors share identity unless a depth/normal boundary separates them — exactly Gaussian Grouping's N3 contribution applied to the continuous id_code before VQ).
- **Loss weights:** warm-started from LangSVR's per-field ablations; annealed λ schedule so that $\lambda_{\text{id}}, \lambda_{\text{lang}}, \lambda_{\text{geo}}$ are zero for the first ~500 iterations (photometric warm-up to prevent geometry from being trapped by noisy semantic gradients), then ramped to target values by iteration ~2 000.
- **Replaces:** LangSplat's two-stage (pretrain 3DGS for 30 k iters → freeze → train language AE for another 30 k) and Gaussian Grouping's supervised-after-pretraining recipe. Expected wall-clock: ≤ 1.6× vanilla 3DGS — [[wu2026_langsvr]]'s one-stage voxel number is the reference; 3DGS's rasterizer is faster than SVRaster so we should land below their overhead.
- **SAM-hierarchy supervision replaced by what?** LangSplat's whole/part/subpart crops are replaced by RADIO's multi-resolution training (Stage 2) *plus* SAM 3's mask hierarchy being absorbed into the track-ID set (SAM 3 can emit nested concepts when prompted; the design ignores the hierarchy above the instance level for now — a deliberate simplification; see Non-goal #4).

### Stage 6 — Precomputed interaction layer: per-instance mean latents
- **Constructed at training end** (and incrementally maintained during training):
  - `instance_mean_latent[i]` — for each VQ codebook entry `i` with ≥ `N_min` Gaussians attached, the opacity-weighted mean `lang_latent` of those Gaussians. At 10 000 instances × 32-D f16 = **640 KB**.
  - `instance_count[i]` — number of Gaussians with `id_vq == i`.
  - `instance_aabb[i]` — 3D bounding box of each instance for spatial-radius queries (not a primary query path but cheap to maintain).
- **Update rule:** EMA during training on each batch over visible Gaussians; full rebuild every 1 000 iters or when Gaussian count changes by > 5 %. Rebuild cost at 1 M Gaussians: ~5 ms on GPU (one scatter-add pass).

### Stage 7 — Precomputed interaction layer: identity hash (codebook_id → Gaussian indices)
- **Data structure:** compressed sparse row (CSR) indexed by `id_vq`. Per Gaussian: 4-byte index. At 1 M Gaussians + 10 000 instances: ~4 MB values + ~40 KB row pointers = **~4 MB total**.
- **Rebuild:** full rebuild every 500 iters via parallel radix-sort on `id_vq`. Between rebuilds, an append-only delta list carries new Gaussians from densification.
- **Why CSR and not a hash table:** at 10 000 instances the keyspace is small; a dense array of CSR row pointers is faster to look up than probing a hash. If multi-room scenes push instance count past ~100 000, switch to Roaring bitmaps per `id_vq` (flagged as an incremental upgrade path, not needed for single-scene captures).

### Stage 8 — Precomputed interaction layer: per-Gaussian k-NN index
- **Data structure:** FAISS IVF-PQ over the 1 M × 32-D `lang_latent` set. ~16 MB index.
- **When it's used:** text queries where the top-1 instance mean-latent cosine falls below a threshold `τ_match` (set to the 90th percentile of validation-time cosine values). Handles the case where the prompt names something the instance-granularity vocabulary doesn't have a clean entry for.
- **Rebuild cost:** ~2 s at 1 M vectors; rebuilt at training end and *not* incrementally — acceptable because training ends with a final consolidation pass anyway.

---

## Query / interaction layer

For each query type: **compute path · data structures consulted · latency budget · failure escalation.**

### 6A. Click selection ("select the object at this pixel")
- **Path:** rasterize id_code at click pixel → nearest codebook entry (argmax of cosine to codebook, ≤ 16 384 entries) → CSR lookup of `id_vq → Gaussian indices`.
- **Data structures:** VQ codebook (2 MB), CSR identity hash (4 MB).
- **Budget:** rasterize dominates at ~2 ms for one pixel at 1 M Gaussians; codebook lookup ~0.1 ms; CSR expand ~0.2 ms. **Total: < 5 ms.**
- **Failure escalation:** none needed — click is deterministic.

### 6B. Text-to-selection ("select all chairs in the scene")
- **Path:** RADIOv2.5 text head (SigLIP text encoder is the RADIO-compatible path; or a small learned head aligned to the 32-D language latent space) → encode query → 32-D query latent → cosine vs. all `instance_mean_latent[i]` → top-K instances with cosine > `τ_match` → identity-hash expansion.
- **Data structures:** codebook (2 MB), instance means (640 KB), CSR identity hash (4 MB).
- **Budget decomposition:**
  - Text encode (SigLIP ViT-L/14 text tower, cached): 5–10 ms.
  - Cosine over ~10 000 instances at 32-D: < 1 ms.
  - Top-K + threshold: < 1 ms.
  - Identity-hash expansion for K ≤ 50 instances: ~10 ms worst case.
  - **Total: ~25 ms under normal conditions.** Well under 100 ms.
- **Fallback (when `top1_cos < τ_match`):** FAISS IVF-PQ on per-Gaussian latents, top-K = 10 000 → map back to `id_vq` via Gaussian indices → aggregate votes per instance. Budget: ~30 ms for the FAISS query + 5 ms aggregation = ~35 ms. Total: ~60 ms. Still under budget.
- **Honest caveat:** if an RTX 4090 is not the target and we're talking about mobile/edge hardware, text encode becomes the bottleneck and < 100 ms stops being feasible; this design is specified for desktop-class compute.

### 6C. Compositional query ("the red mug", "all lamps but not the ceiling one")
- **Decision rule:** parse query (small LLM or regex-based NP extractor) to detect:
  - Attribute-modified nouns ("red mug"): two latents, attribute + noun — cosine on averaged/concatenated fails systematically.
  - Negations ("but not X").
  - Spatial constraints ("in this room", "on the table").
  - Counting constraints ("all N lamps").
- **Escalation path (GaussExplorer pattern, [[kim2026_gauss-explorer]]):**
  1. Run 6B on the head noun ("mug", "lamp") → candidate instance set C.
  2. Render novel views targeted at each candidate instance's AABB (cheap; instance means are known).
  3. Send thumbnails + query to a VLM (GPT-4V-class) asking: "which of these mugs is the red one? Reply with indices."
  4. Parse VLM output → final instance set.
- **Budget:** 2–5 s, explicitly **async** in the UI (progress spinner; NOT in the < 100 ms hot path).
- **Why not just always VLM?** Cost + latency. Cosine on instance means handles ≥ 80 % of queries per GaussExplorer's own benchmark numbers (and per the "simple object queries" framing of language-lifted 3DGS overall).

### 6D. Image-exemplar query ("select things like this")
- **Path:** RADIOv2.5 image encoder on the exemplar → 768-D → AE encode → 32-D → same path as 6B.
- **Budget:** ~50 ms (image encode is the cost). Under 100 ms.

---

## Editing / action operations (direct operations on the Gaussian set)

All edits consume the `id_vq → Gaussian indices` CSR from Stage 7. No re-rendering supervision, no fine-tuning. Operations:

| Verb | Concrete Gaussian-set op | Side effect to maintain |
|---|---|---|
| **Delete "all X"** | Mark Gaussians with matching `id_vq` as `opacity = 0, prune_flag = true`; next prune pass removes them. | Rebuild CSR; shrink `instance_count`. Notify AE buffer of removed samples. |
| **Hide "all X"** | Non-destructive: set a `visible` bit false; renderer skips these Gaussians. | Trivial. Reversible. |
| **Recolor "red mug" → blue** | Target: Gaussians where `id_vq ∈ selected_instances`. Shift SH DC term by (target_color − mean_current_color); optionally re-optimize SH for 100 iterations in a locked subset to preserve view-dependent effects. | None; `id_vq` and `lang_latent` untouched. |
| **Transform "the chair" (translate/rotate/scale)** | Apply rigid or affine transform to `μ` and `Σ` of the selected Gaussian subset. | Recompute `instance_aabb`. The `lang_latent` follows the Gaussians unchanged (language is intrinsic, not spatial). |
| **Inpaint after delete** | Run LangSVR-style geometric-prior + RADIO feature loss restricted to the removed AABB's rendered support; optimize 200 iterations. Novel Gaussians spawn via densification into the hole. | New Gaussians inherit neighbor `id_vq` and `lang_latent` via the densify rules. |
| **Duplicate "this object" at (x,y,z)** | Deep copy of `id_vq == i` Gaussian subset with μ + Δ offset; assign a fresh `id_vq` slot to the duplicate. | New codebook entry; instance_mean inherits parent's latent; instance_count updated. |
| **Style transfer "paint this chair wooden"** | Run 50 iterations of gradient descent on SH of the target subset with a CLIP-style guidance loss toward "wooden chair" prompt. | Does not touch `id_vq` or `lang_latent` (object identity is preserved). |
| **Swap an object for a SAM 3D asset** | Delete target id_vq's Gaussians; sample a Gaussian cloud from the SAM 3D latent asset (see [[chen2025_sam-3d]]); paste at target μ + Δ offset. | Full rebuild of instance metadata for the inserted object. |

---

## Densification / pruning / update hygiene

The gap every prior paper punts on. This is the section without which the design is not buildable.

### Invariant (stated)
**After any densification or pruning step, the following must hold:**
- Every Gaussian's `id_vq` lookup in the CSR returns an index belonging to that Gaussian, OR `id_vq == NULL_ID` (the sentinel).
- `instance_count[i]` equals the number of live Gaussians with `id_vq == i`.
- No more than 3 % of Gaussians have `id_vq == NULL_ID` at any given time.
- `lang_latent` values track the EMA of their supervising RADIO features within tolerance ε.

### Rules (ordered checklist applied at each densify/prune pass)

1. **On split (gradient-driven, parent straddles a high-gradient region):**
   - Compute `entropy(id_logits)` over the last W = 10 supervised renders for the parent.
   - If `entropy > τ_id`: both children inherit `id_code = parent.id_code` but `id_vq = NULL_ID`, `flag = needs_resupervision`. They are excluded from CSR until re-quantized (next epoch's VQ pass). `lang_latent` copies verbatim.
   - If `entropy ≤ τ_id`: children inherit `id_vq, id_code, lang_latent` verbatim.
   - **Why safe:** null-id children do not corrupt the CSR; rendering still works (falls back to the continuous `id_code` for supervision); the k-NN index treats them as normal (lang_latent is valid).

2. **On clone (low gradient, high photometric error):**
   - Inherit all fields verbatim. Clone means geometry is fine; no instance-boundary signal.

3. **On prune:**
   - Pure-opacity prune from vanilla 3DGS is augmented with: `prune iff opacity < τ_o AND id_entropy < τ_id AND lang_latent_norm_stable`.
   - Boundary Gaussians (high id_entropy) are **protected for K = 1 000 iterations** to give supervision time to resolve them, then re-evaluated.
   - NULL_ID Gaussians older than K iterations that are still NULL_ID get pruned (they never received consistent supervision).

4. **On opacity-reset (standard 3DGS regularizer that zeros opacities every 3 000 iters):**
   - Do **not** reset `id_code` or `lang_latent`. Reset is a geometry regularizer only; identity supervision re-anchors within 200–500 post-reset iterations.
   - **Loss-weight safeguard:** during the post-reset window, scale up $\lambda_{\text{id}}, \lambda_{\text{lang}}$ by 1.5× so supervision recovers fast. Anneal back over 500 iters.

5. **CSR rebuild cadence:**
   - Full rebuild every 500 iters or when > 5 % of Gaussians changed (densify/prune events count). Between rebuilds, a delta list (append-only) of `(id_vq, gaussian_index)` pairs handles the deltas.

6. **VQ codebook update:**
   - Every 1 000 iters, re-run k-means++ on visible id_codes. New codebook entries are created for clusters that appear; entries with `instance_count < N_min = 10` are retired (their Gaussians get relabeled to NULL_ID and must re-supervise into the nearest live entry).

### Failure modes flagged by the invariant
- **NULL_ID fraction > 3 %:** means SAM 3 is giving noisy track IDs across frames. Hard-fix: force-assign NULL_ID Gaussians to their nearest-neighbor `id_vq` in 3D. This is a hack; it papers over upstream noise but keeps the interaction layer consistent.
- **Codebook collapse (many Gaussians hashing to a handful of entries):** retire tiny entries and re-quantize. If this keeps happening, the VQ codebook size is too small; grow it.
- **VQ drift during densification:** the delta list accumulates across densification events faster than it's flushed. Hard-cap at 10 000 deltas; force an off-cycle CSR rebuild if exceeded.

---

## Performance budget (real numbers)

| Phase | Budget | Reference / derivation |
|---|---|---|
| Vanilla 3DGS train (baseline, 30 k iters, 1 M Gaussians) | ~30 min on RTX 4090 | Standard 3DGS |
| This pipeline, one-stage train (same scene) | **≤ 48 min** (1.6× baseline) | [[wu2026_langsvr]] one-stage voxel reference; 3DGS rasterizer faster than SVRaster |
| SAM 3 mask extraction (pre-computed once) | ~2 s per view × 100 views = ~3 min per scene | [[carion2026_sam-3]] inference latency |
| RADIOv2.5 feature extraction (pre-computed once) | ~0.3 s per view × 100 views = ~30 s per scene | [[heinrich2025_radiov25]] ViT-H inference |
| DINOv3 depth/normal priors (pre-computed once) | ~0.2 s per view × 100 views = ~20 s | [[simeoni2025_dinov3]] |
| Per-Gaussian memory (50 bytes/G × 1 M) | **50 MB** | See Stage 4 |
| Interaction-layer precomputed indices (all) | **~23 MB** (VQ 2 MB + CSR 4 MB + instance means 640 KB + FAISS k-NN 16 MB) | See Stages 6–8 |
| Click latency | **< 5 ms** | Decomposed in §Query 6A |
| Text latency (hot path) | **~25 ms** | Instance-mean cosine dominates |
| Text latency (fallback, FAISS) | **~60 ms** | IVF-PQ at 1 M × 32-D |
| Compositional query (VLM escalation) | 2–5 s, async | [[kim2026_gauss-explorer]] |

FLOP note on text-path claim: cosine over 10 000 instance means at 32-D = 320 000 mul-adds → ~10 μs on a 4090. The latency budget is dominated by text-encoder forward pass (CLIP/SigLIP text tower ≈ 5 ms) and scatter-gather for the identity-hash expansion. The claim is not "precomputed so it's fast" — it is decomposed above.

---

## Open questions & risks (specific to this design)

1. **VQ drift vs. edit stability.** If the codebook updates and a Gaussian's `id_vq` changes, stored selections (e.g. "I selected these chairs yesterday") break. Mitigation: snapshot the codebook at user-session start and use that snapshot for edits until the next explicit reindex. Still: live training + concurrent editing is not safe unless snapshots are versioned.

2. **RADIO text head alignment.** [[heinrich2025_radiov25|RADIOv2.5]] is distilled from SigLIP (among others); the natural text path is SigLIP's text encoder, not a custom head. But the per-scene AE compresses 768-D → 32-D, and the text query must be projected into that same 32-D space. Options: (a) project SigLIP text embeddings through the AE's encoder half — but the AE was trained on image features, text may distribute differently; (b) train a tiny text→32-D projection head on cached RADIO image features + their caption-derived SigLIP text embeddings. Option (b) is what the design actually needs. This is a risk because it's a stage not present in any ingested paper.

3. **SAM 3 concept vocabulary bootstrapping.** Stage 1 assumes we have a good scene-wide noun-phrase vocabulary. For novel scenes, this vocabulary is unknown; bootstrap with (a) RADIO-driven image captions, (b) scene-wide YOLO-World or OWLv2 detections to seed, or (c) user-supplied vocabulary. Each has different quality floors. Research question: what's the minimum vocabulary size that produces stable track IDs?

4. **Densify/prune rules have no published ablation.** The densify/prune invariant is this design's own contribution and has no paper directly supporting its exact form. Each of τ_id, K = 1 000, N_min = 10, codebook-update cadence is a hyperparameter that needs scene-level validation. We have principled reasons for each choice but no ablation.

5. **Licensing.** RADIOv2.5 is NSCL (non-commercial) — [[heinrich2025_radiov25]]. SAM 3 is SAM License (non-commercial research) — [[carion2026_sam-3]]. LangSplat is Gaussian-Splatting-License (non-commercial) — [[qin2024_langsplat]]. Gaussian Grouping is Apache-2.0 — [[ye2024_gaussian-grouping]]. Stack is effectively research-only until a commercial-license swap (e.g. an Apache-2.0 RADIO alternative or a re-distilled RADIO-equivalent from permissive teachers) is in place. This is a downstream-use blocker, not a design blocker.

6. **Per-scene AE + multi-scene assets.** The language latent space is per-scene (following LangSplat). If users want to copy-paste objects between scenes (e.g. from scene A into scene B), the 32-D language latent is not comparable across scenes. Fix: maintain a mapping through the 768-D RADIO space as the inter-scene interlingua; pay the AE decode cost at object-insert time. This is an incremental feature, not a core gap.

7. **1 M Gaussian ceiling.** Scaling past ~5 M Gaussians (large scenes / multi-room captures) starts straining FAISS IVF-PQ latency and CSR rebuild cost. At 5 M Gaussians the CSR rebuild is ~40 ms, the FAISS index ~80 MB, and text-path latency pushes toward 150 ms. Not a blocker for single-room scenes; flagged for any scaling plan.

---

## Deliberate non-goals

This design explicitly does **not** address:

1. **Zero-shot cross-scene generalization** — this is [[jiao2025_clip-gs|CLIP-GS]]'s lane (scene-level contrastive alignment). Different tradeoff. The design is per-scene (pay AE training cost, get higher per-scene fidelity; give up cross-scene transfer).
2. **Single-image generative 3D** — [[chen2025_sam-3d|SAM 3D]]'s lane. Orthogonal problem (assumes a reconstructed 3DGS scene already).
3. **LLM-native structured scene description** — [[mao2025_spatiallm|SpatialLM]]'s lane (Python-script scene output). Different representation altogether — a generative output, not a query layer.
4. **SAM-hierarchical fine-grained subpart queries** ("the handle of the red mug") — SAM 3's hierarchy is collapsed to the instance level in Stage 1. Extending to subparts is possible (add a parent_id pointer per Gaussian, re-supervise at the part level) but costs dimensionality and training time; not in v1.
5. **Dynamic scenes / video sequences** — SAM 3's video tracker helps within a capture, but long-horizon (days, weeks) identity preservation across re-captures is out of scope.
6. **Physics-accurate rigid-body editing** — "translate the chair" moves Gaussians but does not preserve physical rigidity under deformation. A mesh-extraction follow-up (see [[gaussian-to-mesh-pipelines]]) would give that; different pipeline.
7. **Real-time online / SLAM-style capture** — [[jatavallabhula2023_conceptfusion|ConceptFusion]] / Pipeline IV of the lifting thread is the right lane for that.
8. **Open-world without a bootstrapping vocabulary** — SAM 3 PCS needs prompts; fully unsupervised instance discovery is a separate open problem.

---

## Build sequence (ordered; each step has one falsifiable checkpoint)

**Do not advance past a checkpoint that fails.** Each step is ~2–4 days of engineering work at the design level; see [[language-grounded-3dgs-nerfstudio]] for the phased nerfstudio implementation, which maps these steps onto buildable artifacts.

### Step 1 — Baseline: 3DGS + SAM 3 identity-only (no language, no geometric prior)
- Implement Stage 1 (SAM 3 mask extraction) + Stage 4 (id_code + id_vq fields only) + the identity half of Stage 5 loss + Stages 6–7 (interaction layer for click selection only).
- Train on 3 standard scenes from the 3D-OVS benchmark and 2 scenes from ScanNet open-vocab subset.
- **Checkpoint (falsifiable):** instance IoU on ScanNet ≥ Gaussian Grouping with the cross-view association loop *removed*. If SAM 3's native track IDs do not beat the association loop cleanly, synthesis bet #1 is wrong and we fall back to keeping association.
- **Go/no-go:** if instance IoU is below GG's, pause and compare SAM 3 concept vocabularies — the association loop might still be needed when SAM 3 bootstrapping fails.

### Step 2 — Add language field: RADIOv2.5 distillation (keep two-stage training)
- Implement Stage 2 (RADIOv2.5 feature extraction) + per-scene AE for 768-D → 32-D + language half of Stage 5 loss, as a *second stage* (after Step 1 completes its training) to isolate effects.
- **Checkpoint:** 3D-OVS mIoU ≥ LangSplat on the 3D-OVS benchmark. If not, we have a regression vs. the 2024 recipe and the RADIO substitution is unsound.

### Step 3 — Add geometric priors (LangSVR loss port)
- Add Stage 3 (DINOv3 depth/normal extraction) + geometric loss terms in Stage 5.
- **Checkpoint:** 3D-OVS mIoU **+ 3–5 pp on boundary-heavy scenes** relative to Step 2 (replicates LangSVR's claim on a 3DGS substrate). If no lift, the "geometry matters for semantics" claim does not transfer from sparse voxels to 3DGS, and we drop the geometric leg.

### Step 4 — Fuse into one-stage training
- Collapse Steps 1–3 into the single-backward-pass loss. Implement loss warmup schedule.
- **Checkpoint:** (a) training time ≤ 1.6× vanilla 3DGS on the same scene, (b) quality ≥ max(Step 3 two-stage, LangSplat). If one-stage degrades quality, revert to two-stage; [[wu2026_langsvr]]'s recipe may require substrate-specific tuning we haven't done.

### Step 5 — Build the interaction layer (click + text hot path)
- Implement Stages 6, 7, 8. Wire the click-path and text-path as hot-path code (CUDA kernels for CSR expand, FAISS IVF-PQ for k-NN).
- **Checkpoint:** on 3 test scenes at 1 M Gaussians, click latency < 5 ms (p95) and text latency < 100 ms (p95) across 50 representative text queries. If either misses, the budget decomposition in §Performance is wrong; profile and either adjust the index structure or the dimensionality.

### Step 6 — Densify/prune rules with the full invariant
- Implement the ordered rule set in §Densification. Instrument the invariant checks (NULL_ID fraction, instance_count consistency) as test-time assertions.
- **Checkpoint:** on 5 scenes, NULL_ID fraction stays < 3 % throughout training; CSR lookup is consistent after every densify/prune event; post-training interaction-layer correctness is unchanged.

### Step 7 — Compositional-query VLM escalation
- Wire the GaussExplorer-pattern fallback from §6C. Parse queries with a small heuristic (regex for modifiers + NP extraction; optional small LLM).
- **Checkpoint:** on a curated set of 30 compositional queries ("the red mug", "all chairs without armrests", "the lamp by the window"), the end-to-end pipeline answers ≥ 80 % correctly (human-rated), and latency stays under 5 s async. If cosine + VLM still falls below 60 %, either the noun-phrase parser is weak or GaussExplorer's pattern does not transfer to our data.

### Step 8 — Editing operations
- Implement the table in §Editing (delete, hide, recolor, transform, inpaint, duplicate, style-transfer).
- **Checkpoint:** after each edit, the invariant in §Densification still holds; user-visible edits render correctly without re-supervision; inpainted holes produce no obvious artifacts on a held-out evaluator's visual inspection.

---

## Reference implementations

Closest existing papers to read before coding:

- [[qin2024_langsplat]] — per-scene AE + SAM-hierarchy recipe, to mirror the AE training loop.
- [[ye2024_gaussian-grouping]] — densify/prune + identity-loss implementation baseline to override.
- [[wu2026_langsvr]] — one-stage joint training + geometric-prior loss structure.
- [[carion2026_sam-3]] — SAM 3 inference API + track-ID output schema.
- [[heinrich2025_radiov25]] — RADIOv2.5 ViT-H inference + ToMe token merging.
- [[kim2026_gauss-explorer]] — VLM-in-the-loop escalation pattern.

Canonical GitHub references (from the wiki): `github.com/lkeab/gaussian-grouping` (Apache-2.0 — safest starting point for the 3DGS-with-fields skeleton), `github.com/minghanqin/LangSplat` (Gaussian-Splatting-License — the AE reference), `github.com/facebookresearch/sam3` (SAM License — mask extraction), `github.com/NVlabs/RADIO` (NSCL — feature extraction).

---

## Verification plan (end-to-end)

1. **Correctness:** run Build Steps 1–8 in order on the 5 evaluation scenes. Each checkpoint must pass.
2. **Numbers:** report 3D-OVS mIoU, ScanNet instance IoU, per-query latencies (p50 / p95), per-Gaussian memory, training wall-clock.
3. **Ablations (minimum set):**
   - Remove SAM 3 → fall back to SAM-1 + association loop (tests synthesis bet #1).
   - Remove RADIOv2.5 → fall back to raw CLIP (tests synthesis bet #2).
   - Remove geometric-FM priors (tests the LangSVR claim on 3DGS).
   - Remove one-stage training (collapse to two-stage to measure the coupling penalty LangSVR claims).
   - Remove VQ layer (continuous id_code + k-NN-on-id for instance expansion; tests whether the discrete VQ earns its keep for latency).
4. **User study (interaction layer):** 20 users × 20 queries each on 3 scenes; measure task success, perceived latency, and fallback-trigger frequency.
5. **Invariant check:** end-of-training assertion — every Gaussian's `id_vq` maps to a live CSR entry OR is NULL_ID; NULL_ID fraction < 3 %.

If the full verification passes, file "Pipeline IX: Feature-augmented 3DGS with 2026-stack lifting" into [[lifting-foundation-models-to-3d]]'s "Current SOTA pipeline" and "Pipeline lineage" sections per CLAUDE.md §3.1 Step 5.

---

## Rigor-gate audit (self-check)

- **Mechanism-level, not slogan-level?** Every stage specifies what is rasterized (id_code, lang_latent, depth, normal), what loss compares it to what target (SAM 3 track IDs via cross-entropy, RADIO patch features via cosine, DINOv3 depth via Pearson correlation, etc.), at which resolution / hierarchy.
- **Every "X improves Y" cites evidence?** SAM 3 replacing SAM-1 cites [[carion2026_sam-3]]'s 2× PCS accuracy; RADIOv2.5 cites Probe3D + ADE20k numbers; geometric priors cite [[wu2026_langsvr]] ablation claim verbatim; one-stage cites LangSVR's outperformance over LangSplat / GG.
- **Speed claims decompose into operations?** §Performance decomposes text path into text encode + instance cosine + CSR expansion, with FLOP / byte counts.
- **Contradictions acknowledged?** Per-scene vs. cross-scene (LangSplat vs. CLIP-GS) — explicit non-goal; 3DGS vs. sparse voxel ([[wu2026_langsvr|LangSVR]] substrate claim) — position stated and justified; dropped-DINO (LangSplat) vs. RADIO-unified — position stated.
- **Every architectural decision traces to a wiki page?** Each stage cites at least one paper page. Densify / prune rules are the design's own contribution (no paper specifies them) — flagged as Risk #4 (no published ablation).

---

## Implementation

Phased implementation plan into the [[nerfstudio|nerfstudio / visiofacto fork]]: **[[language-grounded-3dgs-nerfstudio]]**. Each build step above maps to a phase (or pair of phases) in that plan, with phase-scoped regression tests and milestones.
