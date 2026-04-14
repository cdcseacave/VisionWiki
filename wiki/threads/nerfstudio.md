---
title: Nerfstudio + gsplat Codebase (Visiofacto Fork)
type: thread
tags: [codebase, nerfstudio, gsplat, visiofacto, 3dgs, implementation]
created: 2026-04-13
updated: 2026-04-13
sources: []
status: draft
---

## Scope

This thread documents the **local fork of nerfstudio** at
`/Users/dancostin/Pro/nerfstudio` and the **gsplat submodule** at
`/Users/dancostin/Pro/gsplat`. The goal is to capture the architectural
knowledge needed to implement new 3DGS research ideas (e.g. CoMe's
confidence + variance losses — see
[[come-integration-nerfstudio|design doc]]) without re-deriving the
codebase layout on every attempt.

This is a **reference thread**, not a literature synthesis. It will drift
as the code evolves; re-verify specific file:line references before acting
on them.

---

## Three-layer architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Trainer (engine/trainer.py)                                 │
│   └─ calls pipeline.train_step() in a loop                   │
├─────────────────────────────────────────────────────────────┤
│  Pipeline (pipelines/base_pipeline.py)                       │
│   ├─ DataManager  →  batch of cameras + GT images            │
│   └─ Model        →  render + compute losses                 │
├─────────────────────────────────────────────────────────────┤
│  Model (models/visiofacto.py extends splatfacto.py)          │
│   ├─ gauss_params: nn.ParameterDict (N Gaussians)            │
│   ├─ get_outputs(camera) → calls gsplat.rasterization()      │
│   ├─ get_loss_dict()     → L1 + SSIM + geometric + splat     │
│   └─ step_post_backward() → densification strategy           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  gsplat (cuda/csrc/*.cu + rendering.py)                      │
│   1. fully_fused_projection  (3D Gaussians → 2D)             │
│   2. isect_tiles / isect_offset_encode  (tile binning)       │
│   3. rasterize_to_pixels   (alpha compositing)               │
└─────────────────────────────────────────────────────────────┘
```

---

## Nerfstudio key files

### Model layer

| File | What it does |
|------|-------------|
| [`models/base_model.py:41-228`] | Abstract `Model` — defines `get_outputs()`, `get_loss_dict()`, `get_param_groups()` |
| [`models/splatfacto.py:85-772`] | Base 3DGS model. Owns `gauss_params: nn.ParameterDict` |
| [`models/visiofacto.py:75-2730`] | **Custom fork** — extends splatfacto with multiview/geometric losses, custom densification |

### Pipeline/training layer

| File | What it does |
|------|-------------|
| `pipelines/base_pipeline.py` | Wires DataManager → Model; orchestrates the train step |
| `engine/trainer.py` | Outer loop. Calls `pipeline.train_step()`, then `model.step_post_backward()` for densification |
| `data/datamanagers/` | Samples cameras + GT images per step |

### Config system

| File | What it does |
|------|-------------|
| `configs/method_configs.py` | Registry mapping `{method_name: (Config, description)}` |
| `configs/base_config.py` | Dataclass-based config root |
| `models/visiofacto.py:75-200` | `VisiofactoModelConfig` — add new hyperparameters here |

Nerfstudio uses [tyro](https://github.com/brentyi/tyro) to auto-generate CLI args from dataclass fields. Adding a field to `VisiofactoModelConfig` gives you a `--pipeline.model.<field>` CLI flag for free.

### Custom components

| File | What it does |
|------|-------------|
| `model_components/visiofacto_densification_strategy.py` | `VisiofactoStrategy` — replaces `DefaultStrategy`; controls split/clone/prune |
| `model_components/losses.py` | Loss function utilities |
| `model_components/lib_bilagrid.py` | Bilateral grid for appearance embedding |

---

## Per-Gaussian state model

All per-Gaussian parameters live in **one `nn.ParameterDict`** on the model,
`self.gauss_params`, initialized at [`splatfacto.py:222-231`] and [`visiofacto.py:700-709`]:

| Key | Shape | Meaning |
|-----|-------|---------|
| `means` | `[N, 3]` | 3D position |
| `scales` | `[N, 3]` | log-space diagonal covariance |
| `quats` | `[N, 4]` | unit quaternion (rotation) |
| `features_dc` | `[N, 3]` | SH degree-0 coefficient |
| `features_rest` | `[N, K-1, 3]` | Higher SH bands |
| `opacities` | `[N, 1]` | logit-space opacity |

### Adding a new per-Gaussian attribute

Follow this four-step pattern (verified against the existing
`opacities`/`features_dc` plumbing):

1. **Initialize** in `gauss_params` dict at [`visiofacto.py:700-709`]
2. **Property accessor** at [`visiofacto.py:803-825`] (returns activated value if needed)
3. **State dict loading** at [`visiofacto.py:827-840`] — handles resizing when checkpoint size ≠ runtime size
4. **Densification hooks** — `VisiofactoStrategy` must handle split/clone/prune of the new tensor. See [`model_components/visiofacto_densification_strategy.py:1-80+`]

The densification strategy is the subtle step — forgetting to update it leads to shape mismatches after the first split iteration.

---

## Loss dispatch in visiofacto

`VisiofactoModel.get_loss_dict()` at [`visiofacto.py:2620-2654`] aggregates losses from three sub-methods, returning a dict consumed by the trainer:

```
get_loss_dict()
├── get_single_image_losses()  [2454]    L1 + SSIM on the rendered RGB
├── get_splat_losses()         [2582]    Scale regularization, opacity penalty
└── get_multiview_losses()     [2502]    Cross-view photometric + geometric
     ├── get_normal_smoothing_loss()      [2364]
     ├── get_geometric_consistency_loss() [1646]
     └── patch_match_loss()               [2128]
```

This is the clean place to register **new loss terms**: add a method, call
it from `get_loss_dict()`, and weight it by a config hyperparameter. The
trainer sums the dict and backprops.

---

## Multiview & cross-view state

Visiofacto's big extension over stock splatfacto is **per-camera neighbor tracking** for multiview losses. This is how you'd hook in cross-view aggregation (relevant for CoMe's color/normal variance):

- Metadata fields `camera_neighbors`, `cameras`, `train_dataset` at [`visiofacto.py:397-416`]
- Fixed neighbor selection at [`visiofacto.py:448`] (`_choose_fixed_neighbors()`)
- Cross-view loop inside `get_multiview_losses()` at [`visiofacto.py:2502`] — projects the current view's Gaussians into each neighbor and computes consistency

If you want to **track per-Gaussian statistics across views** (e.g.,
"how much does Gaussian i contribute to each view it's visible in?"),
this is the loop body to augment. Accumulate into a persistent buffer
(e.g., `self.running_color_sum` of shape `[N, 3]`) indexed by `gaussian_ids`.

---

## gsplat internals

### Public entry point

[`gsplat/rendering.py:170`] — `rasterization()`:

```python
rasterization(
    means:      Tensor,  # [N, 3]
    quats:      Tensor,  # [N, 4]
    scales:     Tensor,  # [N, 3]
    opacities:  Tensor,  # [N]
    colors:     Tensor,  # [N, D] — arbitrary feature channels
    viewmats:   Tensor,  # [C, 4, 4]
    Ks:         Tensor,  # [C, 3, 3]
    width, height,
    ...
) → (render_colors, render_alphas, meta_dict)
```

**Critical insight**: `colors` accepts **up to 32 channels (or unlimited via `channel_chunk`)**. Any arbitrary per-Gaussian scalar (e.g. CoMe confidence) or vector can be rendered by stacking it into this tensor — **no CUDA kernel changes needed**. Per-pixel output is the alpha-blended feature: `feature[pixel] = Σ feature[gauss] · α · T`.

### Forward pipeline stages

1. **Projection** — [`cuda/csrc/fully_fused_projection_fwd.cu`]
   - Input: 3D Gaussians, view+intrinsic matrices
   - Output: `radii, means2d, depths, conics` per Gaussian per view

2. **Tile binning** — [`cuda/csrc/isect_tiles.cu`] + [`isect_offset_encode.cu`]
   - Sorts Gaussians into screen-space tiles for efficient rasterization

3. **Rasterization** — [`cuda/csrc/rasterize_to_pixels_fwd.cu:150-200`]
   - Per-pixel, per-tile, front-to-back alpha compositing
   - Core equation: `vis_i = α_i · T_i` where `α_i = exp(-½ · mahalanobis²)` and `T_i` is cumulative transmittance
   - `color[pixel] = Σ_i color_i · vis_i`

### Meta dict returned

Already includes: `radii`, `means2d`, `depths`, `conics`, `opacities`,
`camera_ids`, `gaussian_ids`, `tile_width/height`, tile offset tables.

**NOT already exported**: per-Gaussian-per-pixel `vis_i` weights, per-Gaussian aggregate stats. Extracting these requires CUDA kernel modification (details in the [[come-integration-nerfstudio|CoMe design doc]]).

### Backward path

Custom `torch.autograd.Function` wrappers in [`gsplat/cuda/_wrapper.py`]. Backward kernel [`rasterize_to_pixels_bwd.cu:150-250`] routes gradients through `vis_i`. Any new attribute added to `colors` inherits the backward path for free; a genuinely new tensor parameter needs a new Function + backward kernel.

---

## The "feature channel trick"

gsplat's design makes **feature rendering nearly free**: treat any scalar you want to alpha-blend as an extra color channel. This is the key extension mechanism and is used in practice by:

- **CoMe confidence**: stack per-Gaussian `γ̃_i` as a 1-channel feature → rasterization gives you `Ĉ` (rendered confidence map) per pixel, alpha-blended, with gradients flowing back.
- **Depth rendering**: same idea with `depth_i` as the feature.
- **Normal rendering**: 3-channel feature with per-Gaussian normals.

**Implication**: most CoMe mechanisms can be implemented **without touching gsplat CUDA code** — only at the nerfstudio Python layer. The exception is per-Gaussian aggregate statistics across pixels (for variance losses), which need a custom kernel or a post-pass Python reduction (see design doc for the trade-off).

---

## Recommended implementation rhythm

For any new 3DGS research idea, this is the suggested path through the codebase:

1. **Add hyperparameters** to `VisiofactoModelConfig` ([`visiofacto.py:75-200`])
2. **Add per-Gaussian parameters** (if needed) via the 4-step pattern above
3. **Render auxiliary outputs** via the feature-channel trick in `get_outputs()` ([`visiofacto.py:1111`])
4. **Add loss computation** as a new method on `VisiofactoModel`, called from `get_loss_dict()`
5. **Adjust densification** in `VisiofactoStrategy` only if per-Gaussian gradients behave differently (CoMe does; see design doc)
6. **Gate by config flag** so the new feature can be A/B tested

---

## Open questions about this codebase

- How stable is the fork vs upstream nerfstudio/splatfacto? Merging upstream changes may conflict with visiofacto's custom loss dispatch.
- The `VisiofactoStrategy` is a significant rewrite of the `DefaultStrategy` densification — is there unit test coverage for its split/clone behavior?
- Are there benchmarks comparing visiofacto to stock splatfacto on the standard 3DGS datasets (Mip-NeRF 360, T&T, DTU)? Would inform whether CoMe-style additions are measurable on top of visiofacto's baseline.

## Related

- [[come-integration-nerfstudio]] — design doc for integrating CoMe into this codebase
- [[radiance-field-evolution]] — broader research context
- [[gaussian-to-mesh-pipelines]] — CoMe fits here
- [CoMe paper](../papers/radl2026_confidence-mesh-3dgs.md) — the paper being implemented
