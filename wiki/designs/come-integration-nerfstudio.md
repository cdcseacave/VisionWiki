---
title: "Design Doc: Integrating CoMe Confidence + Variance Losses into Nerfstudio/Visiofacto"
type: design
tags: [3dgs, mesh-reconstruction, confidence, variance-loss, nerfstudio, gsplat, visiofacto, implementation]
created: 2026-04-13
updated: 2026-04-13
sources: [papers/radl2026_confidence-mesh-3dgs.md]
status: draft
---

## Purpose

Concrete implementation blueprint for two contributions from
[CoMe (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md) into the
local **visiofacto** fork of nerfstudio:

1. **Self-supervised per-Gaussian confidence loss** (`L_conf`) with
   confidence-aware densification
2. **Per-primitive variance reducing losses** (`L_color-var`, `L_normal-var`)

This document is written against the codebase layout documented in the
[[nerfstudio]] thread — re-verify specific file:line anchors before coding.

The SSIM decoupling and appearance model changes are **out of scope** for
this design doc; they are a separate, largely independent contribution and
would warrant their own design note.

---

## Part 1 — Understanding the paper before coding

### 1.1 Confidence — what the paper actually says

Each Gaussian gets an **additional learnable scalar** `γ_i ∈ ℝ`,
initialized to `0`. The activated per-primitive confidence is:

$$\tilde{\gamma}_i = \exp(\gamma_i)$$

so the initial value is `1`. The **rendered confidence map** `Ĉ ∈ ℝ^{H×W}_+` is obtained by standard alpha-blending, treating `γ̃_i` as a feature channel:

$$\hat{C}(\mathbf{r}) = \sum_{i=0}^{N-1} w_i(\mathbf{r}) \cdot \tilde{\gamma}_i$$

where `w_i(r) = α_i · T_i` is the standard 3DGS blending weight.

**The confidence loss replaces the photometric loss**:

$$\mathcal{L}_{\text{conf}} = \mathcal{L}_{\text{rgb}} \cdot \hat{C} - \beta \cdot \log \hat{C}$$

computed **per-pixel** and averaged over the image (the paper presents the formula pointwise; `Ĉ` is a map, so the multiplication is elementwise and the reduction is the standard mean).

Key properties:
- When `Ĉ = 1`: `L_conf = L_rgb` (degenerates to vanilla loss).
- When `Ĉ > 1`: the regularizer `−β·log Ĉ` is negative (preferred), but the photometric loss is amplified → Gaussian commits to fitting perfectly.
- When `Ĉ < 1`: photometric loss is damped, `−β·log Ĉ > 0` penalizes low confidence → balanced trade-off when photometric fit is hard (specularities, foliage).
- **Geometric losses are NOT weighted by confidence** — they keep their full weight. This is what makes confidence "balance photometric vs geometric" by only damping photometric influence.

**Hyperparameters** (from §4, verified from /tmp/come.txt):
- `β = 7.5 × 10⁻²`
- Confidence learning rate: `2.5 × 10⁻⁴`
- `L_conf` enabled from **iteration 500** (start of densification)
- Ablation: `β ∈ [0, 0.2]` tested; degenerate when `β = 0`.

### 1.2 Confidence-aware densification

Because `L_conf` scales with confidence, the per-Gaussian gradient magnitude no longer faithfully signals "this Gaussian needs to split". Unconfident Gaussians naturally have large residuals but shouldn't spawn more of themselves.

The paper divides the split threshold by per-primitive confidence:

$$\bar{\tau}_{\text{grad}} = \frac{\tau_{\text{grad}}}{\tilde{\gamma}_i}$$

A Gaussian splits only if its accumulated 2D-means gradient exceeds `τ̄_grad` (now *larger* when `γ̃_i < 1`). This discourages low-confidence Gaussians from cloning.

### 1.3 Variance losses — what the paper actually says

The insight (§3.4 of the paper): after confidence, the remaining failure mode is **view-dependent appearance baked into geometry** — Gaussians whose color or normal changes dramatically across views are surfacing spurious geometry.

Penalize per-primitive variance across the blending weights of one view:

**Color variance loss**:

$$\mathcal{L}_{\text{color-var}} = \sum_{i=0}^{N-1} w_i(\mathbf{r}) \, \lVert c_i - C(\mathbf{r}) \rVert_2^2$$

where `C(r)` is the rendered color of the pixel and `c_i` is the per-Gaussian SH-evaluated color for that ray direction. The paper shows (footnote after Eq. 12) that this equals the **blended color variance along the ray** — minimizing it forces the composited color to come from a single coherent primitive, not a mixture.

**Normal variance loss**:

$$\mathcal{L}_{\text{normal-var}} = \sum_{i=0}^{N-1} w_i(\mathbf{r}) \, \lVert \mathbf{n}_i - \mathbf{N}(\mathbf{r}) \rVert_2^2 = 1 - \lVert \mathbf{N}(\mathbf{r}) \rVert_2^2$$

where `n_i` is each Gaussian's normal (shortest scale axis) and `N(r)` is the rendered normal. **Important simplification**: because `n_i` are unit vectors and `N = Σ w_i n_i`, the weighted squared-distance collapses to `1 − ||N||²`. No per-primitive enumeration needed — just the rendered normal map, which visiofacto already computes.

**Hyperparameters**:
- `λ_color-var = 5 × 10⁻¹`
- `λ_normal-var = 5 × 10⁻³`

### 1.4 Key implementation implications

| Paper mechanism | Needs new CUDA kernel? | Needs new per-Gaussian parameter? | Needs cross-view state? |
|-----------------|------------------------|-----------------------------------|-------------------------|
| Rendered confidence `Ĉ` | **No** — feature-channel trick | Yes (`γ_i`) | No |
| `L_conf` loss | No | — | No |
| Confidence-aware densification | No | — | No |
| `L_normal-var` | **No** — closed-form `1 − ||N||²` | No | No |
| `L_color-var` | **Yes, or Python post-pass** | No | Arguable — single-view is enough |

**Headline conclusion**: only the **color variance loss** may require kernel-level work. Everything else can be added purely in Python on top of visiofacto.

---

## Part 2 — Implementation plan

### 2.1 Phase 0 — Plumbing

**Goal**: no functional changes, just config flags and dead code paths
ready to receive the new losses.

#### 2.1.1 Config additions

Modify [`models/visiofacto.py:75-200`] (`VisiofactoModelConfig`):

```python
# CoMe confidence
enable_confidence: bool = False
confidence_beta: float = 7.5e-2
confidence_lr: float = 2.5e-4
confidence_start_iter: int = 500

# CoMe variance losses
enable_color_variance_loss: bool = False
enable_normal_variance_loss: bool = False
lambda_color_var: float = 5e-1
lambda_normal_var: float = 5e-3
```

Default-off behind feature flags so every addition can be A/B tested
independently.

#### 2.1.2 Register a new method name (optional but recommended)

Add `"visiofacto-come"` to [`configs/method_configs.py`] as a variant that
enables all the CoMe flags by default. This keeps the base visiofacto
config untouched and gives experimenters a one-word preset.

### 2.2 Phase 1 — Confidence parameter plumbing

**Goal**: add `γ_i` to the Gaussian state, make it renderable via gsplat's
feature-channel mechanism, verify `Ĉ = 1` everywhere before densification.

#### 2.2.1 Add `γ` to `gauss_params`

At [`visiofacto.py:700-709`]:

```python
# only if self.config.enable_confidence
"confidence_logit": torch.nn.Parameter(
    torch.zeros(num_points, 1, device=self.device)
),
```

Store the logit (`γ`), not the activated value, so the optimizer sees an
unconstrained parameter.

#### 2.2.2 Property accessor

At [`visiofacto.py:803-825`]:

```python
@property
def confidence(self) -> Tensor:
    """Activated per-Gaussian confidence γ̃_i = exp(γ_i). Shape [N, 1]."""
    return torch.exp(self.gauss_params["confidence_logit"])
```

#### 2.2.3 State dict loading

At [`visiofacto.py:827-840`], handle the new tensor in the resize-on-load
logic. Use the same `torch.nn.Parameter(torch.zeros(newN, 1))` fallback as
the paper's initialization when loading checkpoints of different sizes.

#### 2.2.4 Register with optimizer

At [`base_model.get_param_groups()`] (overridden in visiofacto), add:

```python
if self.config.enable_confidence:
    param_groups["confidence"] = [self.gauss_params["confidence_logit"]]
```

Use `self.config.confidence_lr = 2.5e-4` in the optimizer registration.

#### 2.2.5 Densification hook: split/clone/prune must preserve shape

In [`model_components/visiofacto_densification_strategy.py`], **every site
that clones, splits, or prunes Gaussian tensors must also process
`confidence_logit`**. This is the most common source of subtle shape-mismatch
bugs.

Recommendation: grep the strategy file for `means` — anywhere `means` is
resized, `confidence_logit` must be resized too. Consider refactoring the
strategy to iterate over `gauss_params.keys()` once, rather than naming
each tensor by hand. This future-proofs every subsequent per-Gaussian
addition.

#### 2.2.6 Sanity test

Before adding the confidence loss, verify:
- Run training with `enable_confidence=True` and zero loss weight on confidence.
- `γ` stays at 0, `γ̃ = 1` everywhere → rendered `Ĉ = 1` everywhere.
- Standard training metrics (PSNR, SSIM) unchanged from baseline.

### 2.3 Phase 2 — Render the confidence map

**Goal**: produce `Ĉ: [H, W, 1]` per view, ready to be consumed by the loss.

#### 2.3.1 Feature-channel trick in `get_outputs()`

At [`visiofacto.py:1111`] (the call site for `gsplat.rasterization`):

**Option A (recommended): second rasterization pass with confidence as the feature.**
Cleanly separated from the color pass, no tensor shape gymnastics, minimal
intrusion. Cost: one extra forward pass per step (projection is shared if
`render_mode="RGB"` is decomposed appropriately, see gsplat's
`channel_chunk` docs).

```python
if self.config.enable_confidence:
    render_conf, _, _ = rasterization(
        means=gaussian_means,
        quats=gaussian_quats,
        scales=gaussian_scales,
        opacities=gaussian_opacities,
        colors=self.confidence.detach().new_tensor(self.confidence).unsqueeze(-1)
            if self.confidence.ndim == 1 else self.confidence,   # [N, 1]
        viewmats=viewmats,
        Ks=Ks,
        width=W, height=H,
        sh_degree=None,          # raw features, not SH
        render_mode="RGB",       # alpha-blend features, no background
        ...
    )
    outputs["confidence_map"] = render_conf.squeeze(-1)  # [H, W]
```

**Option B: concat confidence as extra channels on the main `colors` tensor.**
Slightly more efficient (one pass) but couples the color/confidence code.
Not recommended until Option A is shown to be a measurable cost.

#### 2.3.2 Numerical safety

Clamp `Ĉ` to a small minimum (e.g., `1e-6`) before passing to `log` in
the loss. The paper initialization (`γ = 0 → γ̃ = 1`) plus the regularizer
`−β log Ĉ` both bias `Ĉ` away from 0, but densification edge cases can
produce pixels with vanishing alpha.

### 2.4 Phase 3 — The confidence loss (with per-pixel SSIM)

**Goal**: replace `L_rgb` with `L_conf` in `get_single_image_losses()`.

#### 2.4.1 Why SSIM must be per-pixel (where the paper says so)

The paper never adds a standalone "use the SSIM map" sentence — the
requirement is implied by the composition of two equations:

- **Eq. 4** defines `L_rgb = (1 − λ_rgb) · L_1(I_gt, I_render) + λ_rgb · L_D-SSIM(I_gt, I_render)`, with `λ_rgb = 0.2`.
- **Eq. 9** multiplies this `L_rgb` elementwise by `Ĉ ∈ ℝ^{H×W}_+`: `L_conf = L_rgb · Ĉ − β log Ĉ`.

The elementwise product is only well-defined if `L_rgb` is a `[H, W]`
map. Since `L_rgb` is defined by Eq. 4, **both `L_1` and `L_D-SSIM` must
be maps of the same shape**. Using a scalar D-SSIM silently broadcasts
but computes a mathematically different loss.

#### 2.4.2 Full CoMe photometric loss (per-pixel form)

Let `r` index pixels.

**L1 map:**

$$L_1(r) = \frac{1}{3} \sum_{k \in \{R,G,B\}} \left| I_{\text{gt}}^{(k)}(r) - I_{\text{render}}^{(k)}(r) \right|$$

**D-SSIM map** — standard sliding Gaussian window (11×11, σ=1.5), same
as scalar SSIM but returned *before* the final spatial mean:

$$L_{\text{D-SSIM}}(r) = 1 - \text{SSIM\_map}(I_{\text{gt}}, I_{\text{render}})(r)$$

**Per-pixel `L_rgb`:**

$$L_{\text{rgb}}(r) = (1 - \lambda_{\text{rgb}}) \cdot L_1(r) + \lambda_{\text{rgb}} \cdot L_{\text{D-SSIM}}(r), \qquad \lambda_{\text{rgb}} = 0.2$$

**Final scalar confidence loss** (pixel-mean):

$$\mathcal{L}_{\text{conf}} = \frac{1}{HW} \sum_{r} \left[ L_{\text{rgb}}(r) \cdot \hat{C}(r) - \beta \log \hat{C}(r) \right], \qquad \beta = 7.5 \times 10^{-2}$$

If the SSIM-decoupled appearance model (§3.2 of the paper) is also
enabled, replace `L_D-SSIM(r)` with the decoupled variant:

$$L_{\text{D-SSIM}}^{\text{dec}}(r) = 1 - l(I_{\text{gt}}, \hat{I}_{\text{app}})(r) \cdot c(I_{\text{gt}}, I_{\text{render}})(r) \cdot s(I_{\text{gt}}, I_{\text{render}})(r)$$

where `l, c, s` are the three SSIM component maps from the same sliding
window — returned separately instead of being multiplied before the mean.

#### 2.4.3 Implementation

Write one helper that returns the SSIM map, then compose. Put the helper
in `model_components/losses.py`:

```python
def ssim_map(pred: Tensor, gt: Tensor,
             window_size: int = 11, sigma: float = 1.5) -> Tensor:
    """Returns [H, W] per-pixel SSIM via the standard sliding Gaussian
    window. Invariant: ssim_map(...).mean() must match the baseline
    scalar SSIM to floating-point tolerance — use the identical
    window/sigma/padding as the existing scalar SSIM helper."""
    ...

def l_rgb_map(pred: Tensor, gt: Tensor, lam: float = 0.2) -> Tensor:
    l1  = (pred - gt).abs().mean(dim=-1)            # [H, W]
    dss = 1.0 - ssim_map(pred, gt)                  # [H, W]
    return (1 - lam) * l1 + lam * dss               # [H, W]
```

Then in `get_single_image_losses()` at [`visiofacto.py:2454`]:

```python
m = l_rgb_map(rendered_rgb, gt_rgb, self.config.lambda_rgb)   # [H, W]

if (
    self.config.enable_confidence
    and self.step >= self.config.confidence_start_iter
):
    c_hat = outputs["confidence_map"].clamp(min=1e-6)          # [H, W]
    loss_dict["rgb"] = (
        m * c_hat - self.config.confidence_beta * torch.log(c_hat)
    ).mean()
else:
    loss_dict["rgb"] = m.mean()
```

#### 2.4.4 The one invariant that catches ~every bug

With `enable_confidence=True` but the step gate not yet satisfied (so
`Ĉ` is not used), the `else` branch produces `m.mean()`. This **must**
equal the baseline scalar `L_rgb` to floating-point tolerance. If it
doesn't, your `ssim_map` uses a different window, sigma, or padding
convention than the existing scalar SSIM — fix that before trusting any
CoMe training run, because every downstream CoMe metric compounds this
error.

If visiofacto's existing SSIM helper computes the map internally (most
implementations do — they apply the Gaussian window then call `.mean()`
at the end), the cleanest refactor is to split that function in two:
`ssim_map(...)` returning the tensor, and a thin `ssim(...) =
ssim_map(...).mean()` wrapper for all existing call sites that want a
scalar. One-time refactor, no behavior change, and every future
per-pixel weighting scheme (not just CoMe) benefits.

### 2.5 Phase 4 — Confidence-aware densification

**Goal**: divide the split gradient threshold by per-primitive confidence.

In [`model_components/visiofacto_densification_strategy.py`], find the
point where the accumulated 2D-means gradient is compared against `τ_grad`
(usually called something like `grad_threshold` or `densify_grad_threshold`).

Replace:
```python
# before
should_split = (grad_norm > self.grad_threshold)
```
with:
```python
# after
if self.config.enable_confidence:
    conf = model.confidence.squeeze(-1).detach()   # [N], γ̃_i
    per_gauss_threshold = self.grad_threshold / conf.clamp(min=1e-3)
else:
    per_gauss_threshold = self.grad_threshold
should_split = (grad_norm > per_gauss_threshold)
```

Clamp `conf` to avoid blowing up thresholds for pathologically low
confidence. `1e-3` is a safe default — unconfident enough to never
split, not `inf`.

### 2.6 Phase 5 — Normal variance loss

**Goal**: add `L_normal-var = 1 − ||N(r)||²` averaged over pixels.

Visiofacto already renders normal maps for `get_normal_smoothing_loss()`
at [`visiofacto.py:2364`], so the rendered normal `N: [H, W, 3]` is
available.

Add a new method `get_normal_variance_loss()` on the model:

```python
def get_normal_variance_loss(self, rendered_normal: Tensor) -> Tensor:
    """CoMe L_normal-var = 1 - ||N||^2, per-pixel, meaned.
    rendered_normal: [H, W, 3] — alpha-blended unit-ish normals.
    """
    norm_sq = (rendered_normal ** 2).sum(dim=-1)   # [H, W]
    return (1.0 - norm_sq).mean()
```

Call from `get_loss_dict()` at [`visiofacto.py:2620-2654`]:

```python
if self.config.enable_normal_variance_loss:
    loss_dict["normal_var"] = (
        self.config.lambda_normal_var
        * self.get_normal_variance_loss(outputs["normal"])
    )
```

Note: the visiofacto rendered normal uses `depth_to_normal` from
[`gsplat.utils`]; confirm its normalization convention before assuming
`||n_i|| = 1`. If normals are already guaranteed unit length, the
closed form holds exactly; otherwise, normalize per-pixel first.

Note: the normal variance loss seems to smooth too much the estimated surface; experiment with disabling it sooner towards the end of the training, to allow the now settled splats to move onto the fine details of the surface.

### 2.7 Phase 6 — Color variance loss

**Goal**: `L_color-var = Σ w_i(r) ||c_i − C(r)||²`, per pixel, averaged.

This is the only contribution requiring a decision between:

#### Option A — Closed-form equivalent via second moments (pure Python)

The weighted sum of squared deviations equals:

$$\sum_i w_i \lVert c_i - C \rVert^2 = \sum_i w_i \lVert c_i \rVert^2 - \lVert C \rVert^2$$

If you can render **`Σ w_i ||c_i||² `** (second-moment map), the loss is a
one-liner: `(second_moment_map - (C**2).sum(-1)).mean()`.

Rendering `||c_i||²` per Gaussian as an extra 1-channel feature is trivial
via the feature-channel trick:

```python
ci_sq = (per_gauss_evaluated_color ** 2).sum(dim=-1, keepdim=True)  # [N, 1]
# rasterize ci_sq → second_moment_map [H, W]
```

**Subtlety**: `c_i` is the *ray-direction-dependent* SH evaluation, not the
stored SH coefficients. You need to evaluate SH per-Gaussian per-view, then
render. gsplat already does SH evaluation inside rasterization for the
color pass; the cleanest way is to evaluate SH once in Python, pass both
`c_i` and `||c_i||²` as features to a single rasterization call (using
the 32-channel trick: 3 for color, 1 for `||c||²`, plus normals/confidence).

**This is the recommended option** — it avoids any kernel work.

#### Option B — Custom CUDA kernel that accumulates `Σ w_i ||c_i − C||²` directly

Cleaner semantically but requires a new kernel and backward pass. Defer
unless Option A's multi-channel rasterization is shown to be slow.

#### 2.7.1 Implementation (Option A)

1. **Evaluate SH in Python** once per training step for the active camera:
   ```python
   per_gauss_colors = sh_to_rgb(self.features_dc, self.features_rest, dirs)
   # [N, 3]
   ```
2. **Stack into the feature tensor** passed to `rasterization`:
   ```python
   features = torch.cat([
       per_gauss_colors,                        # [N, 3] — the normal color pass
       (per_gauss_colors ** 2).sum(-1, keepdim=True),  # [N, 1]
   ], dim=-1)                                   # [N, 4]
   rendered = rasterization(..., colors=features, sh_degree=None, ...)
   C = rendered[..., :3]                        # [H, W, 3]
   c_sq_map = rendered[..., 3:4]                # [H, W, 1]
   ```
3. **Loss**:
   ```python
   def get_color_variance_loss(self, C, c_sq_map):
       variance = c_sq_map.squeeze(-1) - (C ** 2).sum(-1)
       return variance.clamp(min=0).mean()   # clamp avoids negative from
                                             # floating-point error
   ```

The clamp to `≥ 0` is defensive: mathematically `Σ w_i ||c_i||² ≥ ||C||²` by Jensen's inequality with weights summing to ≤ 1, but floating-point reductions in the rasterizer can produce tiny negatives.

---

## Part 3 — Integration strategy

### 3.1 Ablation-friendly stacking order

Enable in this order to isolate each contribution's effect:

1. Baseline visiofacto (current behavior) — measure F1/PSNR
2. `+ enable_confidence` (no variance) — expect PSNR ≈ same, F1 up, primitive count down ~10–26% per paper
3. `+ enable_normal_variance_loss` — expect F1 up for planar-heavy scenes
4. `+ enable_color_variance_loss` — expect F1 up for specular/high-frequency scenes
5. (separate design) SSIM decoupling

Each step is a gated config flag, so you can flip them independently.

### 3.2 Where this plays nicely with visiofacto's existing custom losses

Visiofacto already has `get_multiview_losses()` (multiview photometric/geometric) and `get_normal_smoothing_loss()` (inter-Gaussian normal consistency). These are **complementary** to CoMe, not redundant:

- Multiview photometric loss: **should be wrapped by confidence** too. Extend Phase 3 to also multiply multiview residuals by `Ĉ_neighbor`.
- Normal smoothing vs normal variance: the former smooths between *adjacent Gaussians in 3D*; the latter aligns *Gaussians that contribute to the same ray*. Both are fine to run together.

### 3.3 Where this may conflict with visiofacto

- **Custom densification strategy**: Phase 4's change must be made in
  `VisiofactoStrategy`, not in gsplat's `DefaultStrategy`. Expect to read
  the strategy carefully — it has custom culling based on
  `cull_sensitivity_base/decay` that may interact with the confidence-based
  threshold. Worth computing per-Gaussian confidence-weighted *culling*
  scores as a stretch goal.
- **Two-stage training**: `enable_geometric_training`, `two_stage_training`,
  `render_geometry_start` at [`visiofacto.py:424-434`] — confirm that CoMe's
  `confidence_start_iter=500` plays well with visiofacto's phase
  scheduling. Likely CoMe should start at `max(500, render_geometry_start)`.

---

## Part 4 — Validation plan

### 4.1 Unit-ish checks

- After Phase 1 (plumbing): checkpoint save/load roundtrips `gauss_params` correctly with the new tensor.
- After Phase 2 (render Ĉ): at iter 0, `Ĉ ≈ 1` everywhere (since `γ̃ = 1` and background alpha ≈ 0 only at silhouettes).
- After Phase 3 (loss): with `β = 0` CoMe should match the ablation in Fig. 7 of the paper (degenerate, no benefit).
- **SSIM map check**: see §2.4.4 — `ssim_map(...).mean()` must match baseline scalar SSIM. Run this as a one-shot assertion on an arbitrary image pair *before* starting any CoMe training.

### 4.2 Reproducibility sanity checks against the paper

| Dataset | Paper metric | Expected with `enable_confidence` + both variance losses |
|---------|--------------|---------------------------------------------------------|
| Tanks & Temples | F1 = 0.521 avg, 20 min | F1 within ±0.02, wall time within 25% |
| ScanNet++ | Best F1 | F1 improvement over baseline visiofacto |
| Mip-NeRF 360 | PSNR within −0.02 dB, −26% primitive count | primitive count should drop noticeably |

Significant deviation would suggest an implementation bug (most likely in
the confidence-weighted photometric computation — verify the reduction
order matches the paper's per-pixel pointwise form).

### 4.3 Qualitative checks (the fastest bug-finder)

Log and visualize:
- **Rendered confidence map `Ĉ`**: should light up on specular highlights, thin foliage, under-observed regions. Paper's Fig. 3 is the reference.
- **Per-Gaussian confidence distribution over training**: should spread from 1 (log-scale).
- **Variance map** (`Σ w_i ||c_i − C||²` per pixel): should be high where multiple Gaussians of different colors compose the same pixel — i.e., where the color variance loss has something to do.

---

## Part 5 — Open questions before implementation

1. **Interaction with visiofacto's multiview losses**: should `Ĉ` also gate the neighbor-view photometric loss in `get_multiview_losses()`? Paper is silent because CoMe is built on vanilla 3DGS, not a multiview-enhanced version. Recommend: yes, by default, with a flag to disable.
2. **SH-evaluation location for Option A color variance**: doing it in Python is simple but duplicates gsplat's internal SH evaluator. Acceptable cost or worth a kernel patch to emit `Σ w_i ||c_i||²` alongside the color?
3. **Per-camera confidence optimizer**: the paper uses a single global LR of `2.5e-4` for all `γ_i`. Should we adopt nerfstudio's
`ExponentialDecayScheduler` like other per-Gaussian params? Probably yes, but start with constant to match the paper.
4. **Normal normalization convention**: visiofacto uses `depth_to_normal`; confirm whether these are unit vectors before assuming the `1 − ||N||²` closed form of `L_normal-var`.

---

## Part 6 — Deliverables checklist

Implementation is complete when:

- [ ] `VisiofactoModelConfig` exposes all 7 new fields with paper defaults
- [ ] `confidence_logit` lives in `gauss_params`, survives save/load, tracks with densification
- [ ] `Ĉ` renders correctly (visualized in eval, not just numeric)
- [ ] `L_conf` replaces `L_rgb` when the flag is on, gated by `confidence_start_iter`
- [ ] `VisiofactoStrategy` divides `τ_grad` by `γ̃_i` when confidence is on
- [ ] `L_normal-var` and `L_color-var` appear in `loss_dict` when their flags are on
- [ ] Ablation sweep (baseline / +conf / +conf+nvar / +conf+all) runs end-to-end on a Tanks & Temples scene
- [ ] Numerical result within ±0.02 F1 of the paper on at least one scene

---

## References

- [CoMe paper (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md) · [arXiv](https://arxiv.org/abs/2603.24725) · [project page](https://r4dl.github.io/CoMe/)
- [[nerfstudio]] — codebase map this design doc is built against
- [SOF paper (Radl 2025)](../papers/radl2025_sof.md) — CoMe inherits the mesh extraction pipeline from SOF
- [[gaussian-to-mesh-pipelines]] — broader context for where CoMe sits
