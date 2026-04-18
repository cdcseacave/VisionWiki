---
title: "Language-Grounded 3DGS — Nerfstudio / Visiofacto Implementation Plan"
type: design
tags: [3dgs, language-grounded, nerfstudio, visiofacto, implementation, phased-plan, sam3, radio, dinov3]
created: 2026-04-15
updated: 2026-04-18
sources:
  - designs/language-grounded-3dgs-2026.md
  - threads/nerfstudio.md
  - designs/come-integration-nerfstudio.md
  - papers/qin2024_langsplat.md
  - papers/ye2024_gaussian-grouping.md
  - papers/carion2026_sam-3.md
  - papers/heinrich2025_radiov25.md
  - papers/simeoni2025_dinov3.md
  - papers/wu2026_langsvr.md
  - papers/kim2026_gauss-explorer.md
realizes_bet: Bet #020   # same as language-grounded-3dgs-2026; this is the implementation plan
realizes_ideas: [[sam3-native-video-ids_carion2026]], [[radiov25-agglomerative-distillation_heinrich2025]], [[dinov3-gram-anchoring_simeoni2025]], [[langsplat-per-scene-autoencoder_qin2024]], [[gaussian-grouping-identity-encoding_ye2024]]
outcome: pending
outcome_date: null
status: draft
---

## Purpose

Concrete phased plan for implementing [[language-grounded-3dgs-2026|the Language-Grounded 3DGS design]] into the local [[nerfstudio|visiofacto fork]] at `/Users/dancostin/Pro/nerfstudio/`. Each phase delivers a **standalone, runnable artifact** with testable behavior, so individual design hypotheses (SAM 3 replaces the association loop; RADIOv2.5 replaces CLIP + DINO + SAM composition; geometric FM priors sharpen semantic boundaries) can be assessed before committing to the fused one-stage pipeline.

**New method name:** `visiofacto-lang` (following the `visiofacto-come` precedent from [[come-integration-nerfstudio]]).

**Implementation pattern:** `VisiofactoLangModel(VisiofactoModel)` subclass + new config + new dataparser / dataset + new densification strategy + offline CLI + viewer extensions. No upstream fork; no gsplat CUDA changes in Phases 0–5 (all rendering uses gsplat's feature-channel trick documented in [[nerfstudio]]).

---

## Context

The approved [[language-grounded-3dgs-2026|design]] adds per-Gaussian identity + language fields to a 3DGS scene, supervised by SAM 3 masks (track-IDs native), RADIOv2.5 patch features, and DINOv3 depth / normal priors, with a precomputed interaction layer hitting click < 5 ms and text < 100 ms at 1 M Gaussians. This plan translates that design into a staged integration into the nerfstudio / visiofacto fork. Each phase is a **standalone, runnable artifact** with testable behavior, so we can assess individual hypotheses from the design (SAM 3 replaces the association loop; RADIOv2.5 replaces CLIP + DINO + SAM composition; geometric FM priors sharpen semantic boundaries) before committing to the fused, one-stage pipeline. The phases are ordered so that **each phase's regression tests keep passing** as later phases land — we never have to "rebuild Phase 3 to ship Phase 5."

---

## Architecture decisions (locked before Phase 0)

1. **Extension style: subclass, not flag.** `VisiofactoLangModel` extends `VisiofactoModel`. Its config `VisiofactoLangModelConfig` extends `VisiofactoModelConfig`. Reason: we are adding 2 per-Gaussian fields, 3 loss terms, a dataparser, a preprocessing CLI, and viewer extensions — too much surface for a flag on the base class. Flags *within* `VisiofactoLangModelConfig` still gate identity / language / geometric legs for A/B testing inside the lang stack.
2. **Method registration: inline.** No pyproject plugin hook exists in this fork (confirmed). Add `"visiofacto-lang"` directly to `nerfstudio/configs/method_configs.py` next to the existing `"visiofacto"` entry at line ~773.
3. **Feature cache: per-view `.npz` sidecar files.** Following the `SemanticDataset` pattern, the offline CLI writes `<dataset>/language_cache/<image_name>.npz` per view containing `sam3_masks`, `sam3_track_ids`, `sam3_presence`, `radio_patch_features`, `depth_prior`, `normal_prior`. The new `LanguageDataset` overrides `get_metadata(data)` (`base_dataset.py:163-164`) to load them.
4. **Per-Gaussian field storage: follow the [[nerfstudio]] thread's 4-step pattern.** (1) init in `gauss_params` dict; (2) property accessor; (3) state-dict resize-on-load; (4) densification-strategy handles split / clone / prune. Verified against CoMe's integration for the `confidence_logit` field.
5. **Feature-channel trick for rasterizing id_code and lang_latent.** gsplat's `rasterization(colors=...)` accepts arbitrary per-Gaussian channels; stack id_code (8-D) and lang_latent (32-D) as additional "color" channels in a second rasterization pass. No CUDA kernel changes needed for Phases 0–5.
6. **VQ codebook lives as a buffer on the model.** `VisiofactoLangModel.id_codebook: Tensor[K, 8]` as a `register_buffer` (not a Parameter — updated out-of-band by k-means, not gradient descent). Persisted with the checkpoint.
7. **Interaction-layer artifacts are post-training.** Phase 6 adds a `ns-language-index` CLI that reads a trained checkpoint and writes `.npz` artifacts (CSR hash, per-instance means, FAISS index). The viewer loads these at session start; training never computes them.
8. **Tests live alongside modules** in `tests/` mirroring the `nerfstudio/` tree. Regression tests use a tiny fixture scene (3–6 views) shipped in `tests/fixtures/language/`.

---

## File layout (end state, after all phases complete)

```
nerfstudio/
  models/
    visiofacto_lang.py                    # NEW: VisiofactoLangModel + Config
  data/
    dataparsers/
      language_scene_dataparser.py        # NEW: wraps ColmapDataParser + language_cache
    datasets/
      language_dataset.py                 # NEW: LanguageDataset.get_metadata()
  model_components/
    visiofacto_lang_losses.py             # NEW: identity CE, lang cosine, geo losses
    visiofacto_lang_strategy.py           # NEW: densify/prune rules with invariants
    vq_codebook.py                        # NEW: online k-means VQ layer + helpers
  scripts/
    language_preprocess.py                # NEW: CLI — SAM3 + RADIO + DINOv3 → .npz
    language_index.py                     # NEW: CLI — trained model → interaction artifacts
    language_eval.py                      # NEW: CLI — 3D-OVS + instance IoU evaluation
  viewer/
    language_viewer.py                    # NEW: click/text/edit ViewerElements
  configs/
    method_configs.py                     # EDIT: add "visiofacto-lang" at line ~773
  tests/
    fixtures/language/                    # NEW: tiny 3-6 view fixture scene
    models/test_visiofacto_lang.py        # NEW
    data/test_language_dataset.py         # NEW
    model_components/test_vq_codebook.py  # NEW
    model_components/test_lang_losses.py  # NEW
    model_components/test_lang_strategy.py # NEW
    scripts/test_language_preprocess.py   # NEW
    scripts/test_language_index.py        # NEW
```

Every phase creates or modifies a precisely-scoped subset of this tree.

---

## Phase summary (at a glance)

| Phase | Purpose | Deliverable | Primary test / assessment | Rough effort |
|---|---|---|---|---|
| **0** | Method scaffolding | `ns-train visiofacto-lang` runs = visiofacto baseline | PSNR/SSIM parity with visiofacto on a standard scene | 1–2 days |
| **1** | Offline preprocessing CLI + dataparser | `.npz` feature caches + `LanguageDataset` loading them into the batch | Cache shapes + overlay visualizations hand-checked on 3 scenes | 1 week |
| **2** | Identity field (no language) | Click-to-select object via viewer; SAM 3 native IDs replace assoc-loop | Instance IoU ≥ Gaussian Grouping on ScanNet open-vocab subset | 1–2 weeks |
| **3** | Language field | Text-to-select via CLI, later viewer | 3D-OVS mIoU ≥ LangSplat on 3D-OVS | 1–2 weeks |
| **4** | Geometric FM priors | Same UX as Phase 3 but with depth / normal regularizers | + 3–5 pp mIoU on boundary-heavy scenes vs. Phase 3 | 1 week |
| **5** | One-stage fusion + full densify/prune invariants | Single-stage `visiofacto-lang` training with invariant assertions | Train time ≤ 1.6× visiofacto; NULL_ID < 3 %; metrics ≥ Phase 4 | 2–3 weeks |
| **6** | Precomputed interaction layer + perf | `ns-language-index` CLI; click / text latency targets hit | Click < 5 ms p95, text < 100 ms p95 at 1 M Gaussians | 1–2 weeks |
| **7** | Editing operations | delete / hide / recolor / transform / duplicate / inpaint / style in viewer | 8 ops produce correct visuals; invariant holds after every edit | 2 weeks |
| **8** | VLM escalation | Compositional queries ("the red mug") routed via VLM | ≥ 80 % accuracy on 30 curated compositional queries | 1 week |

Total: ≈ 12–14 weeks of focused engineering for one person. Phases 2, 3, 4 can be accelerated if Phase 1's preprocessing is solid.

---

## Phase 0 — Method scaffolding

**Purpose:** establish `visiofacto-lang` as a registered method that behaves identically to `visiofacto` (all new feature flags default off). Ship the config dataclass, the model subclass with empty hooks, and regression tests proving parity.

**Files created:**
- `nerfstudio/models/visiofacto_lang.py` — `VisiofactoLangModelConfig(VisiofactoModelConfig)` and `VisiofactoLangModel(VisiofactoModel)`. Config adds flags all default `False`:
  ```python
  enable_identity: bool = False
  enable_language: bool = False
  enable_geometric_prior: bool = False
  one_stage_training: bool = False   # Phase 5
  identity_dim: int = 8
  identity_codebook_size: int = 16384
  language_dim: int = 32
  language_ae_hidden: int = 256
  sam3_cache_dir: Optional[Path] = None
  radio_cache_dir: Optional[Path] = None
  depth_normal_cache_dir: Optional[Path] = None
  lambda_identity: float = 1.0
  lambda_language: float = 0.5
  lambda_geometric: float = 0.1
  lambda_spatial_consistency: float = 0.05
  id_warmup_steps: int = 500
  ```
  Model subclass overrides `populate_modules()` to no-op when all flags off, and `get_loss_dict()` to call `super()` and return early when all flags off.

**Files modified:**
- `nerfstudio/configs/method_configs.py` — add `method_configs["visiofacto-lang"] = TrainerConfig(...)` next to visiofacto (line ~773). Use the existing `FullImageDatamanagerConfig` + `ColmapDataParserConfig` for now; Phase 1 swaps in the language dataparser.

**Tests:**
- `tests/models/test_visiofacto_lang.py::test_parity_with_visiofacto` — train both methods 1 000 iters on the fixture scene (or a standard short-run scene like `poster`); assert PSNR, SSIM, LPIPS within 1e-3 of each other.
- `tests/models/test_visiofacto_lang.py::test_config_flags_register` — instantiate `VisiofactoLangModelConfig` with each new flag and assert default is correctly `False` / sentinel.

**Assessment criteria:**
- `ns-train visiofacto-lang --data <dataset>` completes without errors.
- Parity test: PSNR delta < 0.01 dB vs. `visiofacto` after 1 000 iters on fixture scene. A larger delta means the subclass introduced a subtle non-determinism; fix before advancing.
- CI green on CPU + one GPU backend.

**What Phase 0 deliberately does NOT do:** any actual new field, loss, or preprocessing. This is pure plumbing; the whole point is to prove the subclass is behaviorally transparent.

---

## Phase 1 — Offline preprocessing CLI + LanguageDataset

**Purpose:** produce the cached per-view tensors (SAM 3 masks / track-IDs, RADIOv2.5 patch features, DINOv3 depth + normal) that every training-side phase consumes. Establish the dataparser + dataset pipeline that streams them into the batch. **This is the phase that unblocks all downstream training work.**

**Files created:**
- `nerfstudio/scripts/language_preprocess.py` — CLI: `ns-language-preprocess <dataset> [--force] [--models sam3,radio,dinov3]`.
  - Loads SAM 3 via `github.com/facebookresearch/sam3` (install as a conda / pip dep; non-commercial license flagged).
  - Loads RADIOv2.5 ViT-H from `torch.hub` or `github.com/NVlabs/RADIO` (NSCL license flagged).
  - Loads DINOv3 + a monocular depth/normal head (Depth-Anything-v2 style) from HuggingFace.
  - For each view in the Colmap dataset, runs all three and writes `<dataset>/language_cache/<image_name>.npz` with keys:
    - `sam3_masks: uint8 [H, W]` (track-id per pixel, 0 = background)
    - `sam3_track_ids: int32 [T]` (unique IDs, T = number of tracked concepts)
    - `sam3_concepts: str[T]` (concept string per track)
    - `sam3_presence: float32 [T]` (presence-head logit per track)
    - `radio_patch_features: float16 [H_f, W_f, 768]` (patch features; H_f = H / patch_size)
    - `depth_prior: float16 [H, W]`
    - `normal_prior: float16 [H, W, 3]`
  - Also writes `<dataset>/language_cache/_manifest.json` with model versions, checksum, vocabulary used, and the list of processed images.
- `nerfstudio/data/dataparsers/language_scene_dataparser.py` — `LanguageSceneDataParserConfig(ColmapDataParserConfig)` with `language_cache_dir: Path`. `_generate_dataparser_outputs()` calls `super()` and pokes `language_cache_dir` into `DataparserOutputs.metadata`.
- `nerfstudio/data/datasets/language_dataset.py` — `LanguageDataset(InputDataset)` overriding `get_metadata(data: Dict)`:
  ```python
  def get_metadata(self, data):
      image_name = self.metadata["image_filenames"][data["image_idx"]].stem
      cache_path = self.metadata["language_cache_dir"] / f"{image_name}.npz"
      if not cache_path.exists():
          return {}
      cache = np.load(cache_path)
      return {
          "sam3_masks": torch.from_numpy(cache["sam3_masks"]),
          "sam3_track_ids": torch.from_numpy(cache["sam3_track_ids"]),
          "radio_patch_features": torch.from_numpy(cache["radio_patch_features"].astype(np.float32)),
          "depth_prior": torch.from_numpy(cache["depth_prior"].astype(np.float32)),
          "normal_prior": torch.from_numpy(cache["normal_prior"].astype(np.float32)),
      }
  ```
  Base-dataset merges these into `data` at `base_dataset.py:163-164`; they land in the model's batch via the existing pipeline — no DataManager surgery.

**Files modified:**
- `nerfstudio/configs/method_configs.py` — `method_configs["visiofacto-lang"]` replaces `ColmapDataParserConfig` with `LanguageSceneDataParserConfig`.

**Scripts + visualization:**
- `nerfstudio/scripts/language_preprocess.py --visualize` — emits `<dataset>/language_cache/_viz/<image>.png` with SAM 3 track-IDs overlaid on the input image (one color per track), depth as a heatmap, and normal as RGB. Hand-checkable.

**Tests:**
- `tests/scripts/test_language_preprocess.py::test_roundtrip_fixture` — run the CLI on the 3-view fixture scene; assert all three `.npz` files produced, shapes match expectations, manifest checksums stable.
- `tests/scripts/test_language_preprocess.py::test_idempotent` — re-run the CLI without `--force`; verify it skips already-processed views.
- `tests/data/test_language_dataset.py::test_get_metadata_shapes` — instantiate `LanguageDataset` on fixture; pull one item; assert metadata keys present and shapes correct.
- `tests/data/test_language_dataset.py::test_missing_cache_gracefully` — feed a dataset with no cache; `get_metadata` returns empty dict; downstream code doesn't crash.

**Assessment criteria:**
- Preprocessing a 100-view scene completes in < 10 minutes on RTX 4090 (SAM 3 dominates at ~2 s/view).
- Hand-check 3 visualization outputs: SAM 3 track IDs look coherent across views; depth maps make geometric sense; normals point outward where expected.
- Cache disk usage < 200 MB per 100 views (RADIO features dominate at ~150 MB).
- `ns-train visiofacto-lang` still produces visiofacto-parity PSNR (flags still all off; cache is ignored by the model in Phase 1).

**Dependencies added to the repo:** `sam3`, `nvlabs-radio`, `dinov3` (or equivalent). Licenses flagged in the design's Risk #5 apply; document in `README`.

**What Phase 1 does NOT do:** no new per-Gaussian fields, no new losses. The model still trains as visiofacto-parity; only the data pipeline changes.

---

## Phase 2 — Identity field (no language, no geometry)

**Purpose:** add the `id_code` + `id_vq` per-Gaussian fields and the identity loss supervised by SAM 3 track IDs. Test synthesis bet #1 from the design: *SAM 3's native video track IDs obsolete Gaussian Grouping's IoU + feature cross-view association module.*

**Files created:**
- `nerfstudio/model_components/vq_codebook.py` — `OnlineKMeansVQ` class with:
  - `codebook: Tensor[K, 8]` (register_buffer on the owner module, initialized via k-means++ on the first 10 k visible id_codes).
  - `encode(id_codes: Tensor) -> Tensor[ids]` via nearest-codebook lookup (batched).
  - `update(visible_id_codes: Tensor)` — called every `codebook_update_every` iters; runs k-means++ for 5 iters on a sample of visible id_codes; retires entries with `instance_count < N_min` (relabels their Gaussians to a NULL_ID sentinel = -1).
  - Persistable: `state_dict` / `load_state_dict` handle codebook + frequency table.
- `nerfstudio/model_components/visiofacto_lang_losses.py::identity_loss()`:
  ```python
  def identity_loss(
      rendered_id_logits: Tensor,      # [H, W, K]  (cosine-softmax over codebook)
      sam3_masks: Tensor,              # [H, W]     (track-id per pixel)
      sam3_track_to_codebook: Dict[int, int],  # from scene-level pairing module
  ) -> Tensor:
      # Cross-entropy where each pixel's target is the codebook-index
      # that the scene-level pairing module associated with its SAM 3 track.
      ...
  ```
  The *scene-level pairing module* runs once per few epochs: it collects, for each SAM 3 track, the histogram of codebook entries that Gaussians projecting to that track's mask currently hash to; assigns the most-frequent-non-conflicting codebook entry as that track's target. Prevents codebook-index drift across views from becoming supervision noise.

**Files modified:**
- `nerfstudio/models/visiofacto_lang.py`:
  - `populate_modules()` now initializes `gauss_params["id_code"] = nn.Parameter(torch.zeros(N, 8))` and the VQ codebook buffer when `config.enable_identity`.
  - `@property def id_code(self) -> Tensor: return self.gauss_params["id_code"]`.
  - `get_outputs()` adds a second `rasterization()` call with `colors=self.id_code` (feature-channel trick — no gsplat changes); output stored as `outputs["id_code_map"]`.
  - `get_loss_dict()` calls `identity_loss(...)` when `config.enable_identity` and step > `config.id_warmup_steps`.
  - State-dict resize-on-load updated to include `id_code`.
- `nerfstudio/model_components/visiofacto_lang_strategy.py` — new `VisiofactoLangStrategy(VisiofactoStrategy)` that preserves `id_code` through split / clone / prune (children inherit verbatim for now — full invariant rules deferred to Phase 5). Register it in the config.

**Viewer extension (minimal):**
- `nerfstudio/viewer/language_viewer.py` — `LanguageViewerExtension` adds a single `ViewerButton("Click to select")` + a click-listener (via `viser_server.scene.on_pointer_event`). On click: read `id_code_map` at that pixel → nearest codebook entry → CSR-expansion (built inline for Phase 2; proper precomputed CSR lives in Phase 6) → set all selected Gaussians' SH to highlight-red for 2 s; revert.

**Tests:**
- `tests/model_components/test_vq_codebook.py::test_encode_nearest` — synthetic id_codes + codebook; verify correct nearest-index.
- `tests/model_components/test_vq_codebook.py::test_update_retires_small` — feed codebook with one entry below `N_min`; verify it's retired.
- `tests/model_components/test_lang_losses.py::test_identity_loss_zero_when_match` — perfect prediction ⇒ CE = 0.
- `tests/model_components/test_lang_losses.py::test_identity_loss_gradient_flows` — backward produces non-zero grads on `rendered_id_logits`.
- `tests/model_components/test_lang_strategy.py::test_split_preserves_id_code` — fake split event on a model with id_code; children inherit.
- Integration: train `visiofacto-lang --pipeline.model.enable_identity=True` for 5 000 iters on a small scene; verify click-select isolates an object (hand-check).

**Assessment criteria:**
- **Benchmark:** on 2 scenes from the ScanNet open-vocab subset, instance IoU ≥ Gaussian Grouping baseline *with SAM-1 + association loop removed*. If below, synthesis bet #1 fails and we need to add a fallback association module or re-tune SAM 3's vocabulary.
- Click selection isolates single instances cleanly (visual inspection). Fails if SAM 3's track IDs fragment across views — diagnose via the preprocessing visualizer from Phase 1.
- NULL_ID fraction after training < 5 % (loose threshold for Phase 2; tightened to 3 % in Phase 5).

**What Phase 2 does NOT do:** no language queries, no geometric priors, no full densify / prune invariant rules (just "inherit verbatim" on split / clone). These land in later phases.

---

## Phase 3 — Language field + text query

**Purpose:** add the `lang_latent` per-Gaussian field, the per-scene autoencoder compressing RADIOv2.5's 768-D patch features to 32-D, and the cosine language loss. Test synthesis bet #2: *RADIOv2.5 as a single feature source replaces LangSplat's CLIP-over-SAM-hierarchy loop and LangSplat's dropped-DINO leg.*

**Files created:**
- `nerfstudio/model_components/language_autoencoder.py` — `LanguageAE(nn.Module)`: 768 → 256 → 32 → 256 → 768 MLP with GELU + LayerNorm; reconstruction loss on cached RADIO features. Trained jointly inside the model in Phase 3 (true one-stage fusion is deferred to Phase 5; here it's jointly optimized but on a separate optimizer param group for clean A/B).
- `nerfstudio/model_components/visiofacto_lang_losses.py::language_loss()`:
  ```python
  def language_loss(
      rendered_lang_latent: Tensor,   # [H, W, 32]  (alpha-blended)
      radio_patch_features: Tensor,   # [H_f, W_f, 768]
      ae: LanguageAE,
  ) -> Tensor:
      # Downsample rendered latent to RADIO patch grid (bilinear).
      # Encode RADIO features through AE → target 32-D.
      # Cosine loss between them. Masked to valid (non-empty) pixels.
      ...
  ```
- `nerfstudio/scripts/language_query.py` — CLI tool: `ns-language-query <checkpoint> --text "chair"`. Runs SigLIP text encoder, projects through `LanguageAE`'s encoder half (initial pass; Phase 3 Risk documented in the design as "RADIO text head alignment"), computes cosine vs. per-Gaussian latents, returns top-K Gaussian indices + produces a highlight rendering.

**Files modified:**
- `nerfstudio/models/visiofacto_lang.py`:
  - `populate_modules()` adds `gauss_params["lang_latent"] = nn.Parameter(torch.zeros(N, 32))` and `self.language_ae = LanguageAE(...)` when `config.enable_language`.
  - `get_outputs()` — third rasterization pass (or extended colors tensor) for `lang_latent`.
  - `get_loss_dict()` calls `language_loss` when enabled.
- `nerfstudio/model_components/visiofacto_lang_strategy.py` — children inherit `lang_latent` verbatim on split / clone (full rule set in Phase 5).
- Viewer — add `ViewerText("Query")` + submit button; on submit, runs the same path as `ns-language-query` and highlights matching Gaussians in the live render.

**Tests:**
- `tests/model_components/test_lang_losses.py::test_language_loss_zero_when_match` — perfectly reconstructed latents ⇒ cosine loss = 0.
- `tests/model_components/test_lang_losses.py::test_language_loss_masked` — pixels outside SAM 3 mask contribute zero.
- `tests/model_components/test_lang_autoencoder.py::test_reconstruction` — trained AE on synthetic RADIO features reconstructs within 0.1 cosine.
- Integration: train `visiofacto-lang --enable_identity=True --enable_language=True` for 10 000 iters on a 3D-OVS scene; run text query "chair"; verify rendered highlight isolates chairs.

**Assessment criteria:**
- **Benchmark:** 3D-OVS mIoU ≥ LangSplat on the 3D-OVS benchmark (per design Build Step 2). If below, the RADIOv2.5 substitution is unsound — investigate whether RADIO patch features project cleanly into the per-scene 32-D AE space.
- Text-query UI subjectively usable on at least 5 queries per scene: "chair", "table", "book", "plant", "floor".
- AE reconstruction cosine ≥ 0.9 on held-out RADIO features for the training scene.

**Open risk carried forward:** the text-path alignment (SigLIP-text → 32-D language latent) is an informed guess in Phase 3; design Risk #2 flags it as a stage not in any ingested paper. Phase 3 uses "project SigLIP text through the AE's encoder half" as a starting point; if cosine degrades sharply on text queries vs. image queries, Phase 3 gains an optional "train a small text→32-D projection head on cached RADIO image features + caption-derived SigLIP text embeddings" sub-phase.

**What Phase 3 does NOT do:** no geometric priors, no compositional queries, no precomputed interaction layer (text-query latency in Phase 3 is seconds, not milliseconds).

---

## Phase 4 — Geometric FM priors (LangSVR loss port)

**Purpose:** add depth-correlation + normal pattern-consistency regularizers supervised by the cached DINOv3 priors. Test the design's position that *LangSVR's objective function is substrate-agnostic* — i.e., the "geometry matters for semantics" finding transfers from sparse voxels to 3DGS.

**Files created:** none (loss helpers added to existing module).

**Files modified:**
- `nerfstudio/model_components/visiofacto_lang_losses.py`:
  ```python
  def depth_correlation_loss(rendered_depth, depth_prior, mask):
      # Pearson correlation; penalize 1 - correlation on reliable pixels.
      ...
  def normal_pattern_consistency_loss(rendered_normals, normal_prior, mask):
      # Local inner-product consistency; penalize deviations at mask boundaries.
      ...
  ```
  Visiofacto already renders normals (see [[nerfstudio]]: `visiofacto.py:2364` `get_normal_smoothing_loss`), so `rendered_normals` is available.
- `nerfstudio/models/visiofacto_lang.py::get_loss_dict` — call the two new losses when `config.enable_geometric_prior`, gated by step > `config.id_warmup_steps`.

**Tests:**
- `test_lang_losses.py::test_depth_correlation_perfect` — rendered == prior ⇒ loss = 0.
- `test_lang_losses.py::test_depth_correlation_anti_correlation` — rendered = -prior ⇒ loss = 2.0 (cap).
- `test_lang_losses.py::test_normal_consistency_pattern` — uniform normal map ⇒ zero penalty.
- Integration: train Phase 3 config vs. Phase 4 config (both language-enabled) on 2 boundary-heavy 3D-OVS scenes; compare mIoU.

**Assessment criteria:**
- **Benchmark:** 3D-OVS mIoU on boundary-heavy scenes ≥ Phase 3 + 3 pp (lower bound of design target). If no lift, the LangSVR finding does not transfer from voxels to 3DGS, and we drop the geometric leg from the preset (but keep the code-path optional).
- No regression on non-boundary-heavy scenes (± 1 pp).
- Training time increase vs. Phase 3 ≤ 15 % (two extra renders + two cheap losses).

---

## Phase 5 — One-stage training + full densification / pruning invariants

**Purpose:** collapse training into a single pass (no AE pretrain); implement the complete densify / prune rule set with invariant assertions. This is the largest phase — it's where "works on 3 scenes" becomes "works robustly at scene-scale." The design's load-bearing engineering contribution lives here.

**Files modified:**
- `nerfstudio/models/visiofacto_lang.py`:
  - Add `_loss_schedule` that zeros `λ_id, λ_lang, λ_geo` for the first `id_warmup_steps` iters (photometric warm-up), then ramps to target via linear warmup over the next 1 500 iters.
  - Wire `config.one_stage_training=True` as the preset; document the flag so two-stage can still be A/B-tested by flipping it off.
  - Add per-step invariant assertions (guarded by `config.assert_invariants=True`, default True in CI, False in production):
    - `NULL_ID fraction < 3 %` (hard fail if violated for > 500 iters).
    - `instance_count[i] == Σ 1[id_vq == i]` for all i.
    - `|lang_latent[i] - EMA_target| < ε` for recently-supervised Gaussians.
- `nerfstudio/model_components/visiofacto_lang_strategy.py` — full rule set from [[language-grounded-3dgs-2026]] §Densification:
  1. **On split:** compute `entropy(id_logits)` over last W = 10 supervised renders for the parent; if `> τ_id`, children get `id_vq = NULL_ID` and flag `needs_resupervision`; `lang_latent` copies verbatim.
  2. **On clone:** inherit verbatim.
  3. **On prune:** `opacity < τ_o AND id_entropy < τ_id AND lang_latent_norm_stable`; boundary Gaussians protected for K = 1 000 iters.
  4. **On opacity-reset:** do NOT reset id_code / lang_latent; scale up `λ_id, λ_lang` by 1.5× for 500 iters post-reset.
  5. **CSR rebuild cadence:** every 500 iters or on > 5 % Gaussian count change.
  6. **VQ codebook refresh:** every 1 000 iters; retire `instance_count < 10` entries.
- `nerfstudio/model_components/vq_codebook.py::VQCodebook.maintenance_step()` — ties together codebook update + NULL_ID cleanup + CSR invalidation.

**Files created:**
- `nerfstudio/scripts/language_invariant_check.py` — offline tool: load a checkpoint and run all invariant checks; emit a report. CI runs this on the fixture scene.

**Tests:**
- `test_lang_strategy.py::test_split_high_entropy_nulls_child` — construct parent Gaussian with synthetic high-entropy id logits; verify children get NULL_ID.
- `test_lang_strategy.py::test_split_low_entropy_inherits` — low entropy ⇒ verbatim inheritance.
- `test_lang_strategy.py::test_boundary_protection` — Gaussian with high id_entropy + low opacity is *not* pruned for K iters.
- `test_lang_strategy.py::test_opacity_reset_preserves_fields` — simulate an opacity reset; id_code and lang_latent unchanged.
- `test_vq_codebook.py::test_maintenance_retires_tiny_entries` — set up codebook with one entry below threshold; run maintenance; verify retired + associated Gaussians relabeled NULL_ID.
- `test_vq_codebook.py::test_csr_rebuild_after_densify` — simulate densify event; verify CSR post-rebuild is consistent.
- Integration: full 30 000-iter training run on 3 scenes with `--assert_invariants=True`; training completes without assertion failures.

**Assessment criteria:**
- **Benchmark:** training time ≤ 1.6× visiofacto baseline on the same scene (one-stage budget per design).
- **Invariant:** NULL_ID fraction < 3 % throughout training on all 5 evaluation scenes.
- **Metrics:** 3D-OVS mIoU + instance IoU ≥ max(Phase 4 two-stage, LangSplat) — within 1 pp of Phase 4, *not* a regression. If one-stage regresses, revert to two-stage default (flag remains).
- **Correctness:** post-training invariant-check tool reports all green on all 5 scenes.

**Failure modes & fallbacks:**
- If NULL_ID fraction exceeds 3 % persistently: the force-assign-to-nearest-neighbor hack from [[language-grounded-3dgs-2026]] §Densification kicks in (gated by `config.allow_null_id_hack=True`); flag this as noise and investigate SAM 3 vocabulary quality.
- If codebook collapse (most Gaussians hash to a handful of entries): grow the codebook size (`config.identity_codebook_size=32768`); if that doesn't help, the scene's instance count is genuinely small and the default 16 384 is overkill — tune per-scene.

---

## Phase 6 — Precomputed interaction layer + performance

**Purpose:** build the post-training artifacts (CSR hash, per-instance mean latents, FAISS k-NN index) that make click < 5 ms and text < 100 ms possible. Replace Phase 2's inline CSR expansion with precomputed lookups.

**Files created:**
- `nerfstudio/scripts/language_index.py` — CLI: `ns-language-index <checkpoint> [--rebuild-faiss]`.
  - Loads the trained model.
  - Runs final VQ pass over all Gaussians.
  - Builds:
    - `codebook: Tensor[K, 8]` (already exists as a buffer; saved verbatim).
    - `csr_identity: {row_ptr: int32[K+1], gaussian_indices: int32[N]}` via parallel radix-sort on `id_vq`.
    - `instance_mean_latent: float16[K, 32]` via opacity-weighted scatter-add.
    - `instance_count: int32[K]`.
    - `instance_aabb: float32[K, 6]`.
    - FAISS IVF-PQ index over `lang_latent` (~16 MB at 1 M × 32-D); `nlist=100, m=8, nbits=8`.
  - Writes all to `<checkpoint>/language_index/` as `.npz` + FAISS binary.
- `nerfstudio/viewer/language_viewer.py` (extended):
  - Loads precomputed artifacts on session start.
  - Click path: rasterize id_code pixel → codebook.encode → CSR lookup → highlight. Instrument with wallclock timers; log p50 / p95.
  - Text path: SigLIP text encode (cached encoder) → 32-D via projection head → cosine vs. instance means → top-K threshold → CSR expand. Instrument timers.
  - Fallback: if `top1_cos < τ_match`, FAISS query → aggregate votes per `id_vq` → expand.

**Files modified:**
- `nerfstudio/models/visiofacto_lang.py::write_checkpoint` hook — on final save, call the indexer automatically (unless `config.skip_auto_index=True`).

**CUDA / optimization considerations:**
- CSR expansion is embarrassingly parallel gather; implement as a CUDA kernel if Python scatter-gather exceeds 1 ms at 1 M Gaussians. Prefer `torch.index_select` first; only drop to CUDA if profiling demands.
- FAISS IVF-PQ is the fallback path; not hot path. Targeted in the 30 ms budget.

**Tests:**
- `test_language_index.py::test_csr_invariant_post_build` — after index build, every Gaussian's `id_vq` maps to a CSR row that contains its own index.
- `test_language_index.py::test_instance_mean_matches_scatter` — compute mean via brute force and via the indexer; assert within 1e-5.
- `test_language_index.py::test_faiss_recall` — synthetic latents with known neighbors; FAISS IVF-PQ top-100 recall ≥ 0.95 at `nprobe=10`.
- Perf: `tests/scripts/test_language_query_latency.py` — 50 text queries × 3 scenes at 1 M Gaussians; assert p95 < 100 ms; click p95 < 5 ms. **Hardware-gated** (runs on RTX 4090-class; skipped in CPU CI).

**Assessment criteria:**
- **Latency:** click p95 < 5 ms, text hot-path p95 < 100 ms on RTX 4090 at 1 M Gaussians (per design targets).
- **Correctness:** text-query results match Phase 3 brute-force within top-5 agreement ≥ 0.9.
- **Memory:** precomputed artifacts ≤ 25 MB total at 1 M Gaussians.

---

## Phase 7 — Editing operations

**Purpose:** expose the 8 editing operations from [[language-grounded-3dgs-2026]] §Editing in the viewer, ensure the invariant in §Densification holds after each edit.

**Files modified:**
- `nerfstudio/viewer/language_viewer.py` — add ViewerButtons / dropdowns for each edit verb:
  - **Delete** — mark matching `id_vq`'s Gaussians as `opacity=0, prune_flag=True`; rebuild CSR; shrink `instance_count[i]`.
  - **Hide** — set a `visible` bit false; renderer skips. Reversible.
  - **Recolor** — shift SH DC of selected subset by `(target_color − mean_current_color)`; optional 100-iter SH re-optimization.
  - **Transform** — rigid or affine transform on `μ, Σ` of the subset; recompute `instance_aabb`. `lang_latent` unchanged.
  - **Inpaint** — run LangSVR loss on the removed AABB for 200 iters; densify into the hole; new Gaussians inherit neighbor `id_vq, lang_latent`.
  - **Duplicate** — deep copy with `μ + Δ`; fresh `id_vq` slot; instance means inherit.
  - **Style transfer** — 50 iters of SH gradient descent with CLIP guidance.
  - **Swap for SAM 3D asset** — delete + generative-3D-asset insertion (stubbed; optional since SAM 3D API access is not guaranteed).
- `nerfstudio/scripts/language_edit.py` — scripted version of all edits (for headless / batch workflows).

**Tests:**
- `test_language_edit.py::test_delete_preserves_invariant` — delete an instance; run invariant check; pass.
- `test_language_edit.py::test_hide_reversible` — hide then unhide; rendered output matches original.
- `test_language_edit.py::test_recolor_target_color` — recolor to red; assert mean rendered color on the instance within 10 of target.
- `test_language_edit.py::test_transform_updates_aabb` — translate instance; assert `instance_aabb` updated.
- `test_language_edit.py::test_duplicate_allocates_fresh_id` — duplicate; new instance has a distinct `id_vq` from the source.
- `test_language_edit.py::test_inpaint_fills_hole_no_artifacts` — delete + inpaint; assert PSNR in the filled region ≥ 25 dB on a held-out view.

**Assessment criteria:**
- All 8 edits produce visually-correct output on at least 3 scenes (hand inspection by 2 reviewers).
- Invariant holds after every edit (automated check).
- Edit latency: delete / hide / transform < 50 ms; recolor / duplicate < 100 ms; inpaint < 30 s; style-transfer < 10 s.

**What Phase 7 does NOT do:** physics-accurate rigid-body editing (design non-goal #6); object exchange between scenes (design Risk #6).

---

## Phase 8 — VLM escalation for compositional queries

**Purpose:** wire the GaussExplorer-style VLM fallback for queries that pure instance-mean cosine cannot answer ("the red mug", "all chairs without armrests", negations, spatial constraints).

**Files created:**
- `nerfstudio/model_components/query_parser.py` — regex-based + optional small-LLM NP extractor detecting: attribute-modified nouns, negations, spatial constraints, counting.
- `nerfstudio/scripts/language_vlm_query.py` — escalation pipeline:
  1. Run Phase 6 text path on the head noun → candidate instance set C.
  2. For each c in C: render a novel view targeting `instance_aabb[c]` at a good pose (cheap — instance means + AABB known).
  3. Send thumbnails + original query to a VLM (OpenAI GPT-4V API or a local VLM if available); ask "which of these mugs is the red one? Reply with indices".
  4. Parse VLM response → final instance set → CSR expand → highlight.
- Viewer: `ViewerTextArea` for compositional queries; async execution with progress spinner (2–5 s budget).

**Files modified:**
- `nerfstudio/viewer/language_viewer.py` — wire the compositional-query path behind a "Complex query" toggle on the text input.

**Tests:**
- `test_query_parser.py::test_extract_attribute_modified_noun` — "red mug" ⇒ `(attribute="red", noun="mug")`.
- `test_query_parser.py::test_extract_negation` — "chairs but not the blue ones" ⇒ `(include="chairs", exclude="blue ones")`.
- `test_vlm_query.py::test_candidate_rendering` — given a fake instance set, render thumbnails for each AABB; assert outputs have expected shape.
- `test_vlm_query.py::test_vlm_parse_indices` — feed mock VLM response "1, 3, 4"; parse to `{1, 3, 4}`.
- Integration: curated 30-query benchmark; manually annotated ground truth; automated scoring.

**Assessment criteria:**
- **Benchmark:** ≥ 80 % accuracy on 30 curated compositional queries (human-rated).
- **Latency:** end-to-end ≤ 5 s async; never blocks the viewer main thread.
- **Cost:** one VLM call per query; cache identical queries per-scene.

**Fallback behavior:** if VLM API is unavailable (no credentials, offline), compositional queries gracefully degrade to showing the Phase 6 text-path result for the head noun with a warning banner.

---

## Cross-phase concerns

### Regression test harness
- `tests/fixtures/language/` ships a 3–6-view toy scene with precomputed SAM 3 / RADIO / DINOv3 caches (hand-curated; ~50 MB checked in via git-lfs or dvc).
- CI runs Phase 0 parity test, Phase 1 preprocessing idempotency, Phase 2–5 short-training sanity (500 iters), Phase 6 latency (GPU-gated), Phase 7 edit invariant, Phase 8 parser unit tests.
- Full benchmarks (3D-OVS, ScanNet subset) run on-demand via `make language-benchmarks`; hardware-gated.

### Backward compatibility
- Every phase keeps `ns-train visiofacto-lang --pipeline.model.enable_identity=False --enable_language=False --enable_geometric_prior=False` equivalent to vanilla visiofacto (Phase 0 parity test — kept green across all phases).
- Existing visiofacto checkpoints load into `VisiofactoLangModel` (state_dict resize-on-load handles missing new fields by zero-init).

### Observability
- Every phase emits a `language_metrics.json` summary at training end: `{"null_id_fraction", "codebook_active_entries", "instance_count_mean", "train_time_ratio_vs_visiofacto", "3d_ovs_mIoU", "instance_iou"}`.
- Tensorboard panel for per-Gaussian fields: id_entropy histogram, lang_latent norm distribution, codebook size over time, NULL_ID fraction over time.

### Dependency / license bookkeeping
Document in `README` and `NOTICE`:
- SAM 3 — non-commercial SAM License.
- RADIOv2.5 — NSCL (non-commercial).
- DINOv3 — check DINOv3 license (likely research).
- LangSplat code patterns — Gaussian-Splatting-License (non-commercial).

Flag in CI: any PR adding a new model dep must update `NOTICE` with the license.

### Evaluation datasets
- **3D-OVS** — per-scene language mIoU; download script in `scripts/download_3d_ovs.py`.
- **ScanNet open-vocab subset** — instance IoU; script in `scripts/download_scannet_ov.py`.
- **Custom indoor scenes** (5 captures) — subjective quality + latency; private to the project.

---

## Open engineering questions (resolve during Phase 1–3)

1. **Dataparser memory at scale.** At 1 000-view datasets, loading RADIO features (~150 KB per view) into RAM on every `get_metadata` call may thrash. Options: (a) memory-map the `.npz` files; (b) batched prefetch in the DataManager; (c) move to a single scene-level `.safetensors` or zarr store. Decide during Phase 1 profiling.
2. **RADIO patch resolution.** RADIOv2.5 ViT-H produces patch features at stride 14. 3DGS training views can be 1 080p (77×108 patches). Decision: store at that resolution, bilinear-upsample at training time, or rasterize feature field at patch resolution. Measured during Phase 3.
3. **VQ codebook initialization.** K-means++ on the first 10 000 visible id_codes works when Gaussians have spread out — but at iter 500 they're still near the init. Options: (a) random init to 16 384 unit vectors, first VQ update at iter 2 000; (b) init from SAM 3 track count (one codebook entry per track). Decide in Phase 2.
4. **SigLIP → 32-D text projection.** Design Risk #2. Phase 3 uses "project SigLIP text through the AE's encoder half" as a starter; if cosine degrades vs. image queries, Phase 3 gains a small text projection head.
5. **Viewer real-time integration.** Viser may not support click events with per-frame latency budgets < 5 ms (it's a web-socket protocol). Decide in Phase 2 whether to fall back to a desktop Qt viewer for the interaction layer.

---

## Deliberate non-implementation

Phases exclude:
- **SAM 3D asset insertion (Phase 7 "swap")** — stubbed because SAM 3D inference pipeline + asset bank are not a given. If useful, split into Phase 7b.
- **Cross-scene language transfer** — design non-goal #1. Would need a separate design.
- **Real-time online / SLAM-style capture** — design non-goal #7.
- **Physics-accurate editing** — design non-goal #6.

---

## Milestones & go / no-go gates

| Milestone | Phases | Gate |
|---|---|---|
| **M1 — Foundations stable** | 0, 1 | `ns-train visiofacto-lang` runs, preprocessing CLI produces visualized caches on 3 scenes |
| **M2 — Identity works** | 2 | Instance IoU ≥ Gaussian Grouping without association loop; click-select usable in viewer |
| **M3 — Language works (research validation)** | 3, 4 | 3D-OVS mIoU ≥ LangSplat; geometric-prior lift ≥ 3 pp on boundary scenes |
| **M4 — Production-ready one-stage** | 5 | Invariants hold; training ≤ 1.6× visiofacto; metrics match Phase 4 |
| **M5 — Interactive** | 6, 7 | Click < 5 ms, text < 100 ms, 8 edits work |
| **M6 — Compositional queries** | 8 | ≥ 80 % accuracy on curated benchmark |

Each milestone is a natural stopping point — if resources run out before M6, the project still ships a usable artifact.

---

## Getting started

After this design is approved and you're ready to begin:

1. Kick off Phase 0 by opening two files:
   - `nerfstudio/models/visiofacto_lang.py` (new — skeleton).
   - `nerfstudio/configs/method_configs.py` (add `"visiofacto-lang"` entry at line ~773).
2. Run the Phase 0 parity test on the fixture scene.
3. On green, begin Phase 1 preprocessing CLI.

Cross-refs for implementers:
- [[language-grounded-3dgs-2026]] — research-synthesis design this plan implements.
- [[nerfstudio]] — codebase architecture, 4-step per-Gaussian-field pattern, feature-channel trick.
- [[come-integration-nerfstudio]] — comparable-complexity extension already in this fork; use as a template for config / densification patterns.
