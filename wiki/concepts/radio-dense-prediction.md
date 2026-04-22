---
title: RADIO / C-RADIO dense-prediction heads — segmentation & depth recipes
type: concept
tags: [radio, foundation-model, segmentation, depth-estimation, adaptors, how-to, sam3, dpt, linear-probe]
created: 2026-04-22
updated: 2026-04-22
sources: [wiki/papers/heinrich2025_radiov25.md, wiki/papers/ranzinger2026_c-radiov4.md, wiki/papers/carion2026_sam-3.md]
status: draft
---

A practical reference for using [[heinrich2025_radiov25|RADIOv2.5]] and
[[ranzinger2026_c-radiov4|C-RADIOv4]] outputs to produce segmentation masks
(ideally instance-level, SAM/SAM3-style) and depth maps. Organized by task
rather than by paper. Distinguishes **what ships** from **what you have to
train**.

## TL;DR — what's available off-the-shelf

| Task | Recipe | Training needed? | Best RADIO variant |
|---|---|---|---|
| **Promptable instance segmentation** (SAM3-style, text + box + point prompts) | [sam3-radio fork](https://github.com/mranzinger/sam3-radio) — C-RADIOv4 replaces SAM3's ViT-L+ encoder; SAM3 decoder unchanged | **No** | C-RADIOv4-H or -SO400M |
| **Promptable class-agnostic masks** (SAM1-style, point/box only) | RADIOv2.5 or C-RADIOv3 `sam` adaptor features → frozen SAM decoder (community recipe) | **No** (but no official script) | RADIOv2.5 / C-RADIOv3 |
| **Zero-shot open-vocab semantic segmentation** | [RADSeg](https://github.com/RADSeg-OVSS/RADSeg) — single torch-hub call with class list + `lang_model='siglip2'` | **No** | `c-radio_v3-l` (officially supported) |
| **Zero-shot open-vocab (DIY, simpler)** | `siglip2-g` adaptor spatial features · cosine with SigLIP2 text embeddings · bilinear upsample · argmax | **No** | C-RADIOv4 (best); C-RADIOv3 / RADIOv2.5 with `clip` adaptor |
| **Closed-vocab semantic seg** (ADE20K, Cityscapes, ...) | Linear probe (Probe3D-style) · UPerNet / Mask2Former / DPT head on spatial features | **Yes** (head only; freeze backbone; hours on 1 GPU) | C-RADIOv4-H for best numbers |
| **Monocular depth** | **Train a head** — DPT decoder on multi-layer features · scale-invariant L1 loss | **Yes** (no released depth head) | C-RADIOv4-H (Probe3D-best in family) |
| **Depth without training** | Compose RADIO with a standalone depth model (Depth Anything, Metric3Dv2) — orthogonal forward passes | No RADIO-side training | Any RADIO |

**Headline**: segmentation has zero-training paths (including instance-level for C-RADIOv4 via sam3-radio, and open-vocab via RADSeg). Depth does not — you always train a head, or you use an external depth model alongside.

## Why: the two output modes recapped

One forward pass emits two streams per requested adaptor:

- **`summary`** — `(B, D_summary)`. Global vector. Used for classification / VLM prefix / retrieval.
- **`features`** — `(B, T, D_features)` with `T = (H/patch_size) · (W/patch_size)`, where `patch_size` is 16 (RADIOv2.5 / C-RADIOv3) or 16 (C-RADIOv4). Dense patch tokens. Reshape with `feature_fmt='NCHW'` to get `(B, D_features, H/16, W/16)` — the spatial grid you decode from.

Note that `D_summary ≠ D_features` in general.

When you request teacher adaptors, each one emits its own `(summary, features)` pair projected into that teacher's feature space.

## Adaptor names by RADIO version

From the [NVlabs/RADIO README](https://github.com/NVlabs/RADIO/blob/main/README.md). This matters — the adaptor names change between versions, and not every teacher is exposed in every version.

| Version | Available adaptors | Notes |
|---|---|---|
| AM-RADIO / RADIOv2 | `clip`, `dino_v2`, `sam` | Original three teachers |
| RADIOv2.1 | `clip`, `dino_v2`, `sam` | Same adaptor set |
| RADIOv2.5 | `clip`, `siglip`, `dino_v2`, `sam` | Adds SigLIP v1 |
| C-RADIOv3 | `clip`, `siglip2-g`, `dino_v2`, `sam` | Commercial; SigLIP v1 → SigLIP2-g |
| **C-RADIOv4** | `siglip2-g`, `dino_v3`, `sam3` | Drops DFN-CLIP; upgrades spatial teacher to DINOv3 and mask teacher to SAM3 |

- `clip` / `siglip` / `siglip2-g` adaptors also expose text tokenization — useful for zero-shot open-vocab queries.
- `sam` adaptor outputs are SAM-v1-compatible; `sam3` adaptor outputs feed SAM3's decoder.
- The raw *backbone* outputs (before any adaptor projection) are available too as `vis_output['backbone']`.

## Minimal API

```python
import torch

# Load C-RADIOv4-H with the three teacher adaptors
model = torch.hub.load(
    'NVlabs/RADIO', 'radio_model',
    version='c-radio_v4-h',            # or 'c-radio_v4-so400m', 'radio_v2.5-h', 'c-radio_v3-l'
    progress=True, skip_validation=True,
    adaptor_names=['siglip2-g', 'dino_v3', 'sam3'],
    # vitdet_window_size=8,            # optional: ViTDet windowed inference for latency
)
model.cuda().eval()

# Preprocessing — use the model's own; don't borrow CLIP's
x = model.make_preprocessor_external()(image).cuda()

# Forward (full-grid features — do NOT enable ToMe if you need a spatial grid)
vis_output = model(x, feature_fmt='NCHW')

# Raw backbone (what you usually decode with for geometry / matching)
backbone_summary, backbone_features = vis_output['backbone']

# Per-adaptor outputs
sig2_summary, sig2_features = vis_output['siglip2-g']    # SigLIP2-aligned — text queries
dino_summary, dino_features = vis_output['dino_v3']      # DINOv3-aligned — dense + 3D
sam3_summary, sam3_features = vis_output['sam3']         # SAM3-aligned — mask / instance

# Text embeddings for open-vocab queries (works with siglip2-g and clip adaptors)
sig2_adaptor = model.adaptors['siglip2-g']
text_tokens = sig2_adaptor.encode_text(
    sig2_adaptor.tokenizer(['a cat', 'a car', 'a tree']).cuda(),
    normalize=True,
)   # (C, D_siglip2)
```

Model properties worth knowing:

- `model.patch_size` (16 for the RADIO family).
- `model.max_resolution`, `model.preferred_resolution`.
- `model.window_size` (non-`None` when you enabled ViTDet mode).
- `model.min_resolution_step` — each image dimension must be a multiple of this. With `patch_size=16, window_size=8` → multiple of 128.

## Segmentation recipes

### Recipe S1 — Promptable instance segmentation (SAM3-style, zero training) · **C-RADIOv4 only**

**What you get**: text / box / point-prompted instance masks, essentially matching SAM3's quality on most natural-image domains, *faster* when using ViTDet mode. Fixes SAM3's known `"person"`-query failure (GitHub issue #253).

Use the [mranzinger/sam3-radio](https://github.com/mranzinger/sam3-radio) fork — vision encoder is C-RADIOv4, decoder and prompt-encoder are SAM3's own. API is the same as stock SAM3:

```python
from sam3.model_builder import build_sam3_image_model     # from the sam3-radio fork
from sam3.model.sam3_image_processor import Sam3Processor

model = build_sam3_image_model(version='c-radio_v4-h')    # or c-radio_v4-so400m for speed
processor = Sam3Processor(model)

state = processor.set_image(image)
out = processor.set_text_prompt(state=state, prompt="a bicycle helmet")
masks, boxes, scores = out['masks'], out['boxes'], out['scores']
```

**Known gap**: 10-point cgF1 drop vs. stock SAM3 on SA-Co/Gold, largest on domain-specific queries (`fg_sports_equipment`, `wiki_common`). Prefer stock SAM3 when these domains dominate; prefer sam3-radio when latency dominates or the `"person"` bug matters.

### Recipe S2 — Promptable class-agnostic masks · RADIOv2.5 / C-RADIOv3 (SAM-v1-style)

No officially released script, but the recipe is well-defined: take the `sam` adaptor features and feed them into a frozen SAM-v1 prompt encoder + mask decoder. The community plumbing has to match SAM's expected feature shape (256 channels after a small projection); the RADIO `sam` adaptor is designed to match SAM-v1's image-embedding space.

```python
vis_output = model(x)
sam_features = vis_output['sam'].features              # (B, T, D_sam) ≈ SAM's ViT-H output
# Reshape to SAM decoder's expected (B, 256, 64, 64) and feed into SAM's mask decoder
# with your point / box prompts. No native SAM3-style text concepts in this path.
```

Prefer Recipe S1 on C-RADIOv4 for new work; this recipe remains useful when you're locked to RADIOv2.5 weights or to a frozen SAM-v1 decoder.

### Recipe S3 — Zero-shot open-vocab semantic seg via RADSeg (single call, zero training)

**Best plug-and-play path** for "give me a semantic map for this class list". Paper: [RADSeg (Alama 2025, arXiv 2511.19704)](https://arxiv.org/abs/2511.19704). Code: [RADSeg-OVSS/RADSeg](https://github.com/RADSeg-OVSS/RADSeg). Outperforms previous open-vocab compositions (which stitched CLIP + DINO + SAM) at 2.5× fewer params and 3.95× faster inference.

```python
import torch
model = torch.hub.load(
    'RADSeg-OVSS/RADSeg', 'radseg_encoder',
    model_version='c-radio_v3-l',       # officially validated version
    lang_model='siglip2',
    device='cuda',
    predict=True,
    sam_refinement=False,                # True = RADSeg+ (sharper masks via SAM snap)
    classes=['road','car','sky','building','person', ...],
)
mask = model(image)                      # open-vocab semantic seg
```

**Caveats**: RADSeg is officially validated on `c-radio_v3-l`, not yet on `c-radio_v4`. Upgrading the backbone requires retesting. Output is **semantic**, not instance — pair with S1 (or SAM-refinement via `sam_refinement=True`) if you need instance separation.

### Recipe S4 — Zero-shot open-vocab semantic seg, DIY (no external repo)

The bare-metal version of S3 — no external repo needed. Works on any RADIO with a `clip` or `siglip2-g` adaptor.

```python
vis_output = model(x, feature_fmt='NCHW')
patch_feats = vis_output['siglip2-g'].features              # (B, D, H/16, W/16)

# Text embeddings for each class
classes = ['road', 'car', 'sky', 'building', 'person']
text = sig2_adaptor.encode_text(
    sig2_adaptor.tokenizer([f'a photo of a {c}' for c in classes]).cuda(),
    normalize=True,
)                                                            # (C, D)

# Per-patch cosine similarity, then upsample to image resolution
patch_feats = torch.nn.functional.normalize(patch_feats, dim=1)
logits = torch.einsum('bdhw,cd->bchw', patch_feats, text)    # (B, C, H/16, W/16)
logits = torch.nn.functional.interpolate(logits, size=image.shape[-2:], mode='bilinear')
seg = logits.argmax(1)                                       # (B, H, W)
```

Quality is coarser than RADSeg at the same backbone; boundaries are bilinear-soft. To sharpen: combine with Recipe S1's masks — for each SAM3 mask, take majority-vote CLIP class inside — this is the LangSplat / Trident fallback pattern.

### Recipe S5 — Closed-vocab semantic seg, linear probe (Probe3D-style)

This is how the RADIOv2.5 and C-RADIOv4 papers report their headline mIoU numbers (ADE20k 55.2% for C-RADIOv4-H). Cheapest trainable option.

```python
# Pseudo-code: freeze backbone, train a 1x1 conv
probe = torch.nn.Conv2d(D_backbone, num_classes, kernel_size=1)
# Use backbone features (or dino_v3 adaptor features for pure dense signal)
# Loss: cross-entropy per pixel; upsample logits to target resolution via bilinear
```

Training cost: ~an hour on a single GPU for ADE20K-size datasets. Upsampling from `H/16` is blurry on thin objects — acceptable trade-off for simplicity.

### Recipe S6 — Closed-vocab semantic seg, DPT / UPerNet / Mask2Former (SOTA quality)

When boundary quality matters more than simplicity. Tap features from multiple transformer blocks (DPT pattern) or add a heavier decoder (UPerNet / Mask2Former).

```
layer 6   (B, D, 32, 32) ─┐
layer 12  (B, D, 32, 32) ─┤  DPT reassemble + fusion + upsample → (B, C, H, W)
layer 18  (B, D, 32, 32) ─┤                                        (UPerNet / M2F heads work too)
layer 24  (B, D, 32, 32) ─┘
```

RADIO does not publish a ready DPT head, but the standard [DPT codebase](https://github.com/isl-org/DPT) slots onto any ViT backbone. Freeze RADIO, train the head on ADE20K / Cityscapes. See also the [Probe3D repo](https://github.com/mbanani/probe3d) for the paper's exact training configs.

## Depth recipes

There is **no pretrained depth head for RADIO** published by NVIDIA. The Probe3D depth numbers in the RADIO papers come from training-time linear probes whose weights are not released. Every depth recipe below requires training something, except the last.

### Recipe D1 — Train a linear probe (paper reproduction)

Follow the [Probe3D codebase](https://github.com/mbanani/probe3d) — `train_depth.py` accepts any frozen ViT backbone. Swap in RADIO as the backbone; reuse Probe3D's scale-invariant L1 loss and NYU-v2 / Hypersim / KITTI training splits.

Output: a linear (or shallow MLP) probe on top of `D_backbone` or `dino_v3` adaptor features. RADIOv2.5-H hits 85.7% Probe3D depth; C-RADIOv4-H hits 85.55%. Not SOTA vs. dedicated depth models but decent for a generic frontend.

### Recipe D2 — Train a DPT depth head (sharp boundaries)

Same pattern as Recipe S6, but with a single-channel regression output. The [DPT codebase](https://github.com/isl-org/DPT) has this ready as `DPTDepthModel`. Freeze RADIO, tap multi-layer features, train on a mixed-domain dataset (Hypersim + NYU-v2 + KITTI for indoor+outdoor) with scale-invariant L1 + gradient-matching losses.

Training cost: ~a day on one GPU for a single dataset, several days for mixed-domain.

### Recipe D3 — Compose RADIO with a standalone depth model (zero RADIO-side training)

If you don't *require* depth to come from RADIO features, the simplest path is to run RADIO and a specialized depth model *independently* on the same image:

- [[depthanythingv3|Depth Anything v3]] — feed-forward mono depth, strong on unseen scenes.
- [[metric3dv2|Metric3Dv2]] — metric-scale mono depth + normals.

You get better depth than any RADIO head likely will, at the cost of a second forward pass. Use RADIO for everything *else* (segmentation, matching, VLM prefix), depth from the specialist. This is what [[wu2026_langsvr|LangSVR]]-style pipelines do implicitly in the voxel-online OP of [[lifting-foundation-models-to-3d]].

### Recipe D4 — Fine-tune a pretrained DPT head from another ViT backbone (speculative, not validated)

Published DPT weights (e.g. from Depth Anything) are trained on DINOv2 / DINOv3 features. You could try loading those weights and fine-tuning on RADIO features — feature dimensions and statistics differ, so it's not drop-in, but feature-space adapter layers might bridge the gap faster than training from scratch. No paper has demonstrated this; flagged as a future bet in [[foundation-features-for-geometry]].

## Community integrations & third-party tooling

- **[RADSeg](https://github.com/RADSeg-OVSS/RADSeg)** (Alama 2025) — open-vocab semantic segmentation, torch-hub loadable.
- **[sam3-radio](https://github.com/mranzinger/sam3-radio)** (NVIDIA fork) — C-RADIOv4 as SAM3's vision encoder.
- **[NVLabs_CRADIOV3 FiftyOne plugin](https://github.com/harpreetsahota204/NVLabs_CRADIOV3)** — FiftyOne integration for embedding extraction, attention visualization, similarity search on C-RADIOv3 models. Community plugin (not NVIDIA-official).
- **[Voxel51 blog post on C-RADIOv4](https://voxel51.com/blog/c-radiov4-distilled-vision-foundation-model)** — third-party walkthrough with dataset-scale recipes.

## Gotchas

- **Preprocessing is RADIO-specific**. Call `model.make_preprocessor_external()` — do not reuse CLIP's normalization (`(0.48145466, 0.4578275, 0.40821073)`). The RADIO preprocessor handles per-version differences.
- **Feature channels differ**: `D_summary ≠ D_features`. Further, each adaptor's output dimension differs from the raw backbone's and from other adaptors'. Always index via the adaptor name.
- **Grid layout requires `feature_fmt='NCHW'`**. By default features are returned as `(B, T, D)` in NLC format — convenient for transformer consumers but useless for CNN decoders.
- **Input size must be a multiple of `min_resolution_step`**. With `patch_size=16` alone → multiple of 16. With ViTDet mode and `window_size=8` → multiple of 128. The helper `get_nearest_supported_resolution(H, W)` rounds for you.
- **ToMe kills the grid**. If you enabled Token Merging to fit a VLM budget, the `features` are a set, not a raster — do **not** reshape to `(B, D, H/16, W/16)`. Disable ToMe for any dense-prediction head.
- **C-RADIOv4 drops the `clip` adaptor**. For text queries, use `siglip2-g`. Code that hard-codes `model(x)['clip']` will fail silently on C-RADIOv4.

## What this unlocks for the wiki

- **[[open-vocab-2d-composition]]**: Recipe S3 (RADSeg) and S1 (sam3-radio) are the two SOTA-candidate fillers for op:unified-promptable and op:distilled-single-pass respectively. Bet #021 (Trident over C-RADIOv4 adaptors) is the next compositional step up.
- **[[lifting-foundation-models-to-3d]]**: Recipe S1 (sam3-radio) can replace SAM3 in Bet #013 (Gaussian Grouping with SAM3 native IDs) for a faster, still-native-ID pipeline.
- **[[foundation-features-for-geometry]]**: Recipe D1/D2 depth heads feed the multi-task frontend stage when the same backbone serves geometry + segmentation + depth simultaneously.

## Limitations flagged for future ingest

- **No official depth-head weights anywhere in the RADIO family** as of 2026-04-22. If/when NVIDIA releases a DPT checkpoint, the Recipe D column collapses to zero-training — worth re-checking [NVlabs/RADIO](https://github.com/NVlabs/RADIO) releases at each ingest cycle.
- **RADSeg has not yet validated on C-RADIOv4** (officially supports `c-radio_v3-l`). Upgrade path is straightforward but not published; a RADSeg-v4 announcement would be worth ingesting.
- **Instance-to-semantic bridge** (use SAM3 masks + SigLIP2 text → per-instance label) is a clear composition pattern but not yet a single community utility — each integration currently stitches S1 + S4 by hand.
