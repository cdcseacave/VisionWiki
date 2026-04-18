---
title: "Design Doc: Integrating CoMe Confidence + Variance Losses into Nerfstudio/Visiofacto"
type: design
tags: [3dgs, mesh-reconstruction, confidence, variance-loss, nerfstudio, gsplat, visiofacto, implementation]
created: 2026-04-13
updated: 2026-04-18
sources:
  - papers/radl2026_confidence-mesh-3dgs.md
  - papers/lin2024_vastgaussian.md
realizes_bet: none      # direct paper re-implementation, not a synthesis bet
realizes_ideas: [[per-gaussian-self-supervised-confidence_radl2026]], [[decoupled-appearance-2d-transform_lin2024]]   # idea slugs to be created in Step 6
outcome: pending
outcome_date: null
status: draft
---

## Purpose

Concrete implementation blueprint for two contributions from
[CoMe (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md) into the
local **visiofacto** fork of nerfstudio:

1. **Self-supervised per-Gaussian confidence loss** (`L_conf`) with
   confidence-aware densification (Parts 1–4)
2. **Per-primitive variance reducing losses** (`L_color-var`, `L_normal-var`)
   (Parts 5–6)
3. **Decoupled appearance modeling** — the [VastGaussian
   (Lin 2024)](../papers/lin2024_vastgaussian.md) per-image CNN + embedding
   transformation-map module, with CoMe's SSIM-decoupled refinement (Part 7)

This document is written against the codebase layout documented in the
[[nerfstudio]] thread — re-verify specific file:line anchors before coding.

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

The paper (CoMe Eq. 12) divides the split threshold by per-primitive confidence, **with the divisor clamped from above at 1**:

$$\bar{\tau}_{\text{grad}} = \frac{\tau_{\text{grad}}}{\min(\tilde{\gamma}_i,\; 1)}$$

The upper clamp is the subtle part. Without it, confident Gaussians ($\tilde\gamma_i > 1$) would get an *easier* (smaller) split threshold — the opposite of what we want. Capping the divisor at 1 means the per-primitive threshold is never lower than the base $\tau_{\text{grad}}$: confident Gaussians use the unchanged baseline, and only *low-confidence* Gaussians are discouraged from splitting (their effective threshold rises). This is what the CoMe paper means by "ensure that confident Gaussians are not over-densified".

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

Per CoMe §A.2, clamp `Ĉ` to `[0.001, 5.0]` (both ends matter) before passing to `log` in the loss:

```python
c_hat = c_hat.clamp(min=1e-3, max=5.0)
```

- **Lower clamp (`1e-3`)** is the standard gradient-explosion guard: as `Ĉ → 0`, `∂L_conf/∂Ĉ = L_rgb − β/Ĉ` blows up. CoMe shows the worst-case gradient at `Ĉ = 10⁻³` with `β = 7.5·10⁻²` is ~−75, which is large but survivable.
- **Upper clamp (`5.0`)** is load-bearing and easy to overlook. It prevents over-confident Gaussians from amplifying the photometric loss without bound, which would drive further densification in *already well-reconstructed* regions — the opposite of the intended behavior. CoMe is explicit on this ("preventing further densification of already well-reconstructed regions").

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
    # CoMe §A.2: both ends of the clamp matter (see §2.3.2).
    c_hat = outputs["confidence_map"].clamp(min=1e-3, max=5.0)   # [H, W]
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
# after (CoMe Eq. 12: τ̄ = τ / min(γ̃, 1))
if self.config.enable_confidence:
    conf = model.confidence.squeeze(-1).detach()             # [N], γ̃_i ∈ (0, ∞)
    # Cap divisor at 1 (confident Gaussians use unchanged τ) and floor at
    # 1e-3 (numerical safety for pathologically unconfident primitives).
    divisor = conf.clamp(min=1e-3, max=1.0)
    per_gauss_threshold = self.grad_threshold / divisor      # ≥ τ_grad always
else:
    per_gauss_threshold = self.grad_threshold
should_split = (grad_norm > per_gauss_threshold)
```

Both clamps are load-bearing and independent:

- `max=1.0` comes from the paper — confident Gaussians must not get an easier split.
- `min=1e-3` is a numerical safety floor — prevents the threshold from exploding to $\infty$ if a Gaussian's confidence collapses to zero (which can happen during densification edge cases even though the $-\beta \log \hat C$ regularizer biases away from it).

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

$$\sum_i w_i \lVert c_i - C \rVert^2 = \sum_i w_i \lVert c_i \rVert^2 - \lVert C \rVert^2 \qquad \text{when } C = \sum_i w_i c_i \text{ and } \sum_i w_i \approx 1$$

This is *mathematically equivalent* to CoMe's Eq. 13 ($\sum_i w_i \|\text{sh}(\theta_i, d) - \mathbf{I}\|^2$) under the paper's stated assumption that $\mathbf{I}$ represents the rendered color. The CoMe authors implement this via a custom fused CUDA kernel (§A.3, ~5× faster than naïve PyTorch, with closed-form backward from StopThePop [49]) — but the math is identical. Option A pays the performance cost for a pure-Python implementation without any CUDA work.

If you can render **`Σ w_i ||c_i||² `** (second-moment map), the loss is a one-liner: `(second_moment_map - (C**2).sum(-1)).mean()`.

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

## Part 7 — Decoupled Appearance Modeling (VastGaussian + CoMe SSIM-decoupled)

**Goal.** Port VastGaussian's decoupled appearance module (per-image embedding + CNN → pixel-wise transformation map), apply CoMe's SSIM-decoupled refinement so the module compensates **only for luminance** and cannot mask contrast/structure errors, and compose cleanly with the confidence loss from Phase 3.

The entire Part 7 is additive: rasterization (gsplat) and per-Gaussian state are untouched. Everything happens in a post-rasterization Python module plus a modified loss function.

### 7.1 What the papers actually say

Two references compose here. They must be read together — neither alone specifies the final recipe.

#### VastGaussian (§4.2 + supplement §7)

- Introduce a **per-training-image appearance embedding** $\ell_i \in \mathbb{R}^{64}$. Learnable, one per training camera. Discarded at inference.
- Build a **transformation map** $\mathcal{M}_i \in \mathbb{R}^{H \times W \times 3}$ from the downsampled rendered image plus $\ell_i$:
  1. Downsample $\mathcal{I}_i^r$ by 32× (bilinear) → $\tfrac{H}{32} \times \tfrac{W}{32} \times 3$.
  2. Broadcast $\ell_i$ across spatial dimensions and concatenate → $\tfrac{H}{32} \times \tfrac{W}{32} \times 67$.
  3. Feed to the CNN architecture in Fig. 9 of the supplement (see §7.3 below).
- **Adjusted image** — VastGaussian's original form is a bare elementwise multiply $\mathcal{I}_i^a = \mathcal{I}_i^r \odot \mathcal{M}_i$. Paper's Tab. 6 shows pixel-wise multiplication matches multiplication+addition and multiplication+γ-correction on their datasets. CoMe reimplements this (source was never released) and lands on a different activation — see §7.1.3.
- **Training loss** (VastGaussian's Eq. 3):

  $$
  \mathcal{L}_{\text{app}}^{\text{vast}} = (1-\lambda)\,\mathcal{L}_1(\mathcal{I}_i^a, \mathcal{I}_i) + \lambda\,\mathcal{L}_{\text{D-SSIM}}(\mathcal{I}_i^r, \mathcal{I}_i)
  $$

  — L1 sees the **transformed** render, D-SSIM sees the **untransformed** render. The L1 term learns the appearance delta; the D-SSIM term forces the Gaussians themselves to learn consistent geometry. At inference, $\mathcal{M}_i$ and $\ell_i$ are thrown away.

- **Optimizer**: learning rate $10^{-3}$ for both embedding and CNN (Adam).
- **Training cost (supplement §7.2, Tab. 5)**: w/o appearance module 1h25m and 10.23 GB VRAM; w/ module 1h33m and 11.18 GB VRAM. **+~9% time and +~9% VRAM** for a ~1.7 dB PSNR and +0.027 SSIM lift on Sci-Art. This is the "fast but accurate" budget we inherit — the module is cheap *because of* the 32× downsample (§7.1.3). Skipping the downsample blows the budget on both axes.

#### Why the CNN operates on a 32× downsampled image (not full-res)

This is the single most important design decision and it has two cooperating justifications. Both matter — missing either leads to a wrong implementation.

1. **Efficiency** (the obvious one). The CNN's widest interior tensor is $\tfrac{H}{32} \times \tfrac{W}{32} \times 256$. At 1080p that is ~34 × 60 × 256 × 4 B ≈ 2 MB per image; at 4K it is ~32 MB. Running the same architecture at full resolution would be 1024× more memory and compute at the bottleneck — prohibitive.

2. **Semantics** (the non-obvious but load-bearing one). VastGaussian §4.2 is explicit: the downsampling "prevent[s] the transformation map from learning high-frequency details." If the CNN could represent arbitrary high-frequency per-pixel corrections, it would absorb geometry — edges, texture, parallax — into appearance. The 3D Gaussians would then be free to become geometrically wrong because the 2D transform would hide it. **The downsample is the mechanism that bounds $\mathcal{M}_i$ to the low-frequency content appearance drift actually has** (exposure, white balance, vignetting, coarse color cast).

The upsampling trajectory back to $H \times W$ — four pixel-shuffle blocks plus one bilinear interpolation plus two 3×3 convs — is intentionally low-capacity per-pixel. Each pixel-shuffle stage rearranges channels into spatial dimensions, it does not invent new spatial structure. The final bilinear smooths the output. So even though the *output* is full-resolution, the *effective spatial frequency* of $\mathcal{M}_i$ is bounded by the H/32 input plus the learned interpolation kernels. This is the property that makes the module safe to add.

**Concrete implication for the port**: do **not** "improve" the module by running the CNN at higher resolution, skipping the downsample, or adding residual skip connections from full-resolution input. Every such change breaks the semantic guarantee and reintroduces the geometry-absorption bug the module is supposed to cure.

#### VastGaussian → CoMe §A.1 implementation changes (all adopted here)

CoMe's Sec. A.1 lists five concrete improvements to VastGaussian's module. Adopt all of them — each is cheap and each addresses a known failure mode.

1. **Replace sigmoid with `exp(·)` in Eq. (7)**. CoMe's body text (Eq. 7) writes $\hat{I}^{\text{app}} = \hat{I} \odot \sigma(M_i)$ following GOF's re-implementation; the appendix then replaces $\sigma$ with $\exp$. The reasoning: $\sigma(M) \in (0, 1)$ can only **dim** the render, so the module cannot compensate for *over-exposed* ground truth. $\exp(M) \in (0, \infty)$ handles both directions symmetrically in log-space.

   $$
   \hat{I}_i^{\text{app}} = \hat{I}_i^r \odot \exp(\mathcal{M}_i)
   $$

2. **Zero-initialize the final conv layer** ($W = 0$, $b = 0$). Combined with $\exp$, this gives $\mathcal{M}_i = 0 \Rightarrow \exp(\mathcal{M}_i) = 1$, i.e. the module starts as an **identity transform**. Removes the early-training instability where random CNN outputs multiply the render by an arbitrary field before the rest of the optimizer can react. This is the tiny trick that makes the whole thing trainable from scratch alongside Gaussian densification.

3. **Detach the CNN input** `ds₃₂(Î).detach()`. Prevents gradients of the appearance embedding and CNN from propagating **back into the 3DGS model** through the downsampled-render input path. Without this, the module can "request" the Gaussians to render something specific just to feed its own CNN — a confounded gradient that fights photometric supervision. The render still gets gradients via the normal photometric loss, which is what we want.

4. **Reflection padding instead of center-cropping**. VastGaussian center-crops training images to multiples of 32 so downsampling aligns cleanly; CoMe reflection-pads to the next multiple of 32 instead. Loses no image content and simplifies batched training at mixed resolutions.

5. **Concatenate a lightweight positional encoding** $(u, v, r(u,v))$ with $u, v \in [-1, 1]$ and $r = \sqrt{u^2 + v^2}$. These three channels are appended to `[ds₃₂(Î), ρ_i]`, taking the CNN input from 67 to **70 channels**. $r$ lets the module represent radial effects — **vignetting specifically** — that would otherwise require the network to infer "distance from optical center" purely from the rendered content. CoMe §B.3 argues this is why the CNN approach beats PGSR's 2-scalar exposure model and Hierarchical 3DGS's 3×3 affine: those global mappings cannot compensate for anything spatially varying.

6. **(Perf, optional)** CoMe mentions a custom fused CUDA kernel based on Taming 3DGS [40] that is ~5× faster than the naïve PyTorch implementation of the CNN forward. Out of scope for first integration — reach for it only if the module shows up on profiles.

#### CoMe (§3.2) — SSIM decoupling

VastGaussian's split has a leak: D-SSIM internally fuses luminance, contrast, and structure as $l \cdot c \cdot s$. Because D-SSIM sees the **untransformed** render, **all three SSIM components still try to match the ground truth** — including luminance. But the L1 term and the appearance module are *also* already doing luminance. The result is that the Gaussians still absorb some of the luminance drift through the D-SSIM pathway, which is exactly what VastGaussian was trying to prevent.

CoMe (Eq. 8) fixes this by **decoupling SSIM into its $l \cdot c \cdot s$ factors** and routing each factor to the appropriate image. The paper also gives a second, independent reason for doing this that is worth stating explicitly: **the upsampled $\mathcal{M}_i$ contains CNN artifacts** (pixel-shuffle checkerboarding, bilinear softness at block boundaries). Feeding the transformed render into the $c, s$ maps would pollute the gradient signal the Gaussians see for geometry. Using the **untransformed** render for $c, s$ keeps those artifacts out of the structural/contrast supervision entirely.

$$
\mathcal{L}_{\text{D-SSIM}}^{\text{dec}}(r) \;=\; 1 \;-\; l\!\left(\mathcal{I}_i^a, \mathcal{I}_i\right)(r) \cdot c\!\left(\mathcal{I}_i^r, \mathcal{I}_i\right)(r) \cdot s\!\left(\mathcal{I}_i^r, \mathcal{I}_i\right)(r)
$$

- **Luminance** $l$: runs on the transformed render → the appearance module is allowed to fit luminance.
- **Contrast** $c$, **structure** $s$: run on the untransformed render → the Gaussians must supply these directly, and CNN upsampling artifacts are kept out of the gradient flow.

All three maps come from the **same sliding Gaussian window** (11×11, σ=1.5) that Phase 3's `ssim_map` helper already uses — they are just the intermediate tensors before the final multiplication.

### 7.2 Key implementation implications

| Mechanism | Needs new CUDA kernel? | Needs new per-Gaussian parameter? | Needs per-image state? |
|-----------|------------------------|------------------------------------|------------------------|
| Appearance embedding table $\{\ell_i\}$ | No | No | **Yes** — one [64] vec per training camera |
| Transformation CNN | No | No | No (network is shared across images) |
| `ssim_components(a, b) → (l, c, s)` maps | No | — | — |
| Decoupled D-SSIM | No | — | — |
| Inference-time disablement | No | — | — |

Same headline as the confidence work: **no CUDA changes required**. Everything sits above `rasterization()`.

### 7.3 Phase 7A — Per-image appearance embedding

#### 7.3.1 Mapping training cameras to embedding indices

The embedding table is keyed by **training-camera index**, so every training step needs to know "which embedding do I fetch for this batch?". In nerfstudio, the `RayBundle` / `Cameras` batch carries an `image_idx` field (via the `camera_indices` tensor); visiofacto inherits this. Confirm the field name in your fork (likely `batch["image_idx"]` or `camera.camera_indices`).

Add a method on the datamanager or model to expose `num_train_cameras` at model-construction time. In nerfstudio conventions, the model already receives `num_train_data` through its setup — use that.

#### 7.3.2 `nn.Embedding` table

Add to `VisiofactoModelConfig`:

```python
# Decoupled appearance modeling (VastGaussian + CoMe §3.2 / §A.1)
enable_appearance_module: bool = False
appearance_embedding_dim: int = 64           # VastGaussian ρ_i ∈ R^64
appearance_cnn_width: int = 256              # channels after stem conv
appearance_downsample: int = 32              # hard-coded to match architecture
appearance_activation: Literal["exp", "sigmoid"] = "exp"   # CoMe §A.1 adopts exp
appearance_positional_encoding: bool = True  # CoMe §A.1: (u, v, r) — +3 channels
appearance_detach_cnn_input: bool = True     # CoMe §A.1: detach ds_32(Î)
appearance_pad_mode: Literal["reflect", "center_crop"] = "reflect"  # CoMe §A.1
appearance_lr: float = 1e-3                  # VastGaussian §5.1
appearance_use_ssim_decoupling: bool = True  # CoMe §3.2 refinement
appearance_discard_at_eval: bool = True      # transform is train-only
```

In `VisiofactoModel.__init__()`:

```python
if self.config.enable_appearance_module:
    self.appearance_embed = torch.nn.Embedding(
        num_embeddings=self.num_train_data,
        embedding_dim=self.config.appearance_embedding_dim,
    )
    torch.nn.init.zeros_(self.appearance_embed.weight)
    self.appearance_cnn = AppearanceTransformCNN(
        embed_dim=self.config.appearance_embedding_dim,
        width=self.config.appearance_cnn_width,
        downsample=self.config.appearance_downsample,
    )
```

Initialize embeddings to zero so the network starts as an identity-ish transform — at t=0, the embedding contributes no signal, the CNN's weights are random but small, and training gradually diverges from $\mathcal{M} \approx \text{const}$.

#### 7.3.3 Optimizer registration

Add a new param group in `get_param_groups()`:

```python
if self.config.enable_appearance_module:
    param_groups["appearance"] = list(self.appearance_embed.parameters()) \
                               + list(self.appearance_cnn.parameters())
```

Use `lr=self.config.appearance_lr` with **Adam**, matching VastGaussian §5.1. Do not tie this to the `gauss_params` schedule — the appearance module trains on a different loss surface and wants its own LR.

### 7.4 Phase 7B — The transformation-map CNN

Exact architecture from VastGaussian supplement Fig. 9 with CoMe §A.1's five modifications. Placing this in `model_components/appearance.py`:

```python
class AppearanceTransformCNN(torch.nn.Module):
    """VastGaussian §7.1 + CoMe §A.1: downsampled RGB + broadcast embedding
    (+ optional positional encoding) → log-space transformation map M.

    The final render-side transform is:  Î^app = Î ⊙ exp(M)
    (CoMe §A.1 — exp, not sigmoid; sigmoid cannot compensate over-exposure.)

    Input  : [B, 3 + E + (3 if PE else 0), H/32, W/32]
    Output : M of shape [B, 3, H, W], **zero-initialized** so exp(M) = 1 at t=0.

    Architecture (Fig. 9; block = PixelShuffle → Conv 3x3 → ReLU; channels
    halve per block because PixelShuffle(2) turns C·4 → C while doubling
    spatial, and the Conv keeps channels at half of the stem width):
      stem    : Conv 3x3 : in_ch → 256                     [H/32 × W/32 × 256]
      block 1 : PixelShuffle(2): 256 → 64, 2× spatial
                Conv 3x3       : 64 → 128, ReLU            [H/16 × W/16 × 128]
      block 2 : PixelShuffle(2): 128 → 32, 2× spatial
                Conv 3x3       : 32 → 64,  ReLU            [H/8  × W/8  × 64]
      block 3 : PixelShuffle(2): 64 → 16, 2× spatial
                Conv 3x3       : 16 → 32,  ReLU            [H/4  × W/4  × 32]
      block 4 : PixelShuffle(2): 32 → 8, 2× spatial
                Conv 3x3       : 8  → 16,  ReLU            [H/2  × W/2  × 16]
      upsamp  : Bilinear ×2                                 [H   × W   × 16]
      head    : Conv 3x3 : 16 → 16, ReLU
                Conv 3x3 : 16 → 3                           [H   × W   × 3]

    Final conv weights and bias are zero-initialized (CoMe §A.1) so the
    module starts as identity: M = 0 → exp(M) = 1.
    """

    def __init__(
        self,
        embed_dim: int = 64,
        width: int = 256,
        downsample: int = 32,
        use_positional_encoding: bool = True,
    ):
        super().__init__()
        assert downsample == 32, "architecture hard-codes 5 stages of ×2 = ×32"
        self.use_pe = use_positional_encoding
        in_ch = 3 + embed_dim + (3 if use_positional_encoding else 0)

        self.stem = torch.nn.Conv2d(in_ch, width, kernel_size=3, padding=1)

        # channels halve per block; order is PixelShuffle → Conv → ReLU
        ch = [width, width // 2, width // 4, width // 8, width // 16]  # 256,128,64,32,16
        self.up_blocks = torch.nn.ModuleList()
        for i in range(4):
            c_in_after_shuffle = ch[i] // 4      # PixelShuffle(2) divides channels by 4
            c_out              = ch[i + 1]       # then conv restores to "half of prev"
            self.up_blocks.append(torch.nn.Sequential(
                torch.nn.PixelShuffle(2),
                torch.nn.Conv2d(c_in_after_shuffle, c_out, kernel_size=3, padding=1),
                torch.nn.ReLU(inplace=True),
            ))

        self.upsample = torch.nn.Upsample(
            scale_factor=2, mode="bilinear", align_corners=False,
        )
        # Head's second conv is the one we zero-initialize (§A.1).
        head_hidden = torch.nn.Conv2d(ch[-1], ch[-1], kernel_size=3, padding=1)
        self.head_final = torch.nn.Conv2d(ch[-1], 3, kernel_size=3, padding=1)
        torch.nn.init.zeros_(self.head_final.weight)
        torch.nn.init.zeros_(self.head_final.bias)
        self.head = torch.nn.Sequential(
            head_hidden, torch.nn.ReLU(inplace=True), self.head_final,
        )

    @staticmethod
    def _positional_encoding(h: int, w: int, device, dtype) -> torch.Tensor:
        """CoMe §A.1 PE: (u, v, r) with u, v ∈ [-1, 1], r = sqrt(u² + v²).
        Returns [1, 3, h, w] — spatially broadcasted at downsampled resolution.
        """
        ys = torch.linspace(-1.0, 1.0, h, device=device, dtype=dtype)
        xs = torch.linspace(-1.0, 1.0, w, device=device, dtype=dtype)
        v, u = torch.meshgrid(ys, xs, indexing="ij")
        r = torch.sqrt(u * u + v * v)
        return torch.stack([u, v, r], dim=0).unsqueeze(0)   # [1, 3, h, w]

    def forward(self, rendered_down: torch.Tensor,
                embedding: torch.Tensor) -> torch.Tensor:
        """rendered_down: [B, 3, H/32, W/32]  (caller must .detach() per §A.1)
           embedding:     [B, E]"""
        B, _, h, w = rendered_down.shape
        e = embedding.view(B, -1, 1, 1).expand(-1, -1, h, w)
        feats = [rendered_down, e]
        if self.use_pe:
            pe = self._positional_encoding(h, w, rendered_down.device, rendered_down.dtype)
            feats.append(pe.expand(B, -1, -1, -1))
        x = torch.cat(feats, dim=1)                  # [B, 3+E(+3), H/32, W/32]
        x = self.stem(x)                             # [B, 256, H/32, W/32]
        for blk in self.up_blocks:
            x = blk(x)                               # ends [B, 16, H/2, W/2]
        x = self.upsample(x)                         # [B, 16, H, W]
        return self.head(x)                          # [B, 3, H, W], init 0
```

**Zero-init invariant**: immediately after construction and before any optimizer step, for any input, `forward(...)` must return a tensor of all zeros. Assert this in a unit test — it is the cheapest safeguard against accidental non-zero init creeping back in via a refactor.

**Reflection padding vs exact output size.** Four pixel-shuffle stages plus one bilinear ×2 give exactly 32× spatial upsampling, so when `H, W` are multiples of 32 the output resolution lands exactly right. For non-multiple-of-32 inputs (CoMe §A.1's choice):

- Before downsampling, reflection-pad the render to the next multiple of 32 in each spatial dim: `F.pad(img, pad, mode="reflect")`.
- After the CNN, crop `M_i` back to the original `H × W`.

This is robust to arbitrary aspect ratios and is the pattern to use in visiofacto. Do **not** simply `F.interpolate(..., size=(H, W))` on the final output to "fix" the size — that introduces a nonlinear resampling after the head's zero-init, breaking the identity-at-t=0 guarantee.

### 7.5 Phase 7C — Wiring into `get_outputs()` and the loss

#### 7.5.1 Produce the transformation map

In `VisiofactoModel.get_outputs()`, after the main rasterization call at [`visiofacto.py:1111`]:

```python
rgb_render = outputs["rgb"]          # [H, W, 3], already computed

if (self.config.enable_appearance_module
        and self.training                     # train-only; see 7.5.4
        and "image_idx" in camera_or_batch):
    img_idx = camera_or_batch["image_idx"].view(1)            # [1]
    emb     = self.appearance_embed(img_idx)                   # [1, E]

    # to [1, 3, H, W] for the CNN
    rgb_chw = rgb_render.permute(2, 0, 1).unsqueeze(0)         # [1, 3, H, W]
    H, W = rgb_render.shape[:2]
    d = self.config.appearance_downsample

    # Reflection-pad up to the next multiple of d (CoMe §A.1).
    pad_h = (d - H % d) % d
    pad_w = (d - W % d) % d
    rgb_padded = F.pad(rgb_chw, (0, pad_w, 0, pad_h), mode="reflect")
    Hp, Wp = H + pad_h, W + pad_w

    # Downsample by exactly d, then DETACH (CoMe §A.1) — gradients of the
    # appearance module must not flow back into the 3DGS render.
    rgb_down = F.interpolate(
        rgb_padded, size=(Hp // d, Wp // d),
        mode="bilinear", align_corners=False,
    )
    if self.config.appearance_detach_cnn_input:
        rgb_down = rgb_down.detach()

    M_padded = self.appearance_cnn(rgb_down, emb)              # [1, 3, Hp, Wp]

    # Crop back to original (H, W) — do NOT use F.interpolate here; it would
    # undo the head's zero-init identity guarantee.
    M = M_padded[..., :H, :W].squeeze(0).permute(1, 2, 0)      # [H, W, 3]

    # CoMe §A.1: exp(·), not sigmoid, applied in RGB space.
    if self.config.appearance_activation == "exp":
        rgb_adjusted = rgb_render * torch.exp(M)
    else:  # "sigmoid" (CoMe §3.2 body text; strictly worse than exp)
        rgb_adjusted = rgb_render * torch.sigmoid(M)

    outputs["rgb_adjusted"]   = rgb_adjusted
    outputs["appearance_map"] = M            # raw log-space map (for viz)
```

**Critical**: three details together make this correct, and all three are easy to get wrong silently:

1. **Detach the CNN input.** Without it, the Gaussians receive a second gradient path via "render → downsample → CNN → M → rgb_adjusted → loss" — confounded with the direct photometric path and often destabilizing.
2. **Use `exp`, not sigmoid.** Sigmoid can only dim the render (σ(0)=0.5, and the zero-init would make every initial render half-brightness — you would have to compensate by pre-scaling $\hat I$ by 2, which no one does). Exp is symmetric and plays nicely with zero-init.
3. **Subsequent losses must route correctly.** L1 and SSIM's luminance factor see `rgb_adjusted`; SSIM's contrast and structure factors see `rgb_render` (§7.5.3). Mixing them up silently replicates the original VastGaussian leak.

#### 7.5.2 SSIM-decoupled loss helper

Extend `model_components/losses.py` with a component-wise SSIM that reuses the `ssim_map` infrastructure from Phase 3. The key is that luminance / contrast / structure share the same local mean / variance / covariance computation — it is wasteful and error-prone to compute SSIM twice:

```python
def ssim_components(
    pred: Tensor, gt: Tensor,
    window_size: int = 11, sigma: float = 1.5,
) -> tuple[Tensor, Tensor, Tensor]:
    """Returns per-pixel (l, c, s) maps using the same Gaussian window
    as `ssim_map`. Invariant:
        l * c * s  ==  ssim_map(pred, gt)
    to floating-point tolerance. Verify this in a unit test before trusting
    any SSIM-decoupled training run.
    """
    # μ_x, μ_y, σ_x², σ_y², σ_xy from identical window/padding as ssim_map
    # l = (2 μ_x μ_y + C1) / (μ_x² + μ_y² + C1)
    # c = (2 σ_x σ_y + C2) / (σ_x² + σ_y² + C2)
    # s = (σ_xy + C3)      / (σ_x σ_y  + C3)      with C3 = C2 / 2
    ...

def l_rgb_map_decoupled(
    pred_untransformed: Tensor,    # I_i^r  (for c, s)
    pred_adjusted:      Tensor,    # I_i^a = I_i^r * M
    gt:                 Tensor,
    lam: float = 0.2,
) -> Tensor:
    """CoMe §3.2 per-pixel L_rgb with SSIM-decoupled appearance:
        L_1        on (pred_adjusted, gt)
        l          on (pred_adjusted, gt)
        c, s       on (pred_untransformed, gt)
    """
    l1  = (pred_adjusted - gt).abs().mean(dim=-1)              # [H, W]
    l   = _ssim_luminance(pred_adjusted,      gt)              # [H, W]
    c   = _ssim_contrast (pred_untransformed, gt)              # [H, W]
    s   = _ssim_structure(pred_untransformed, gt)              # [H, W]
    dss = 1.0 - l * c * s                                      # [H, W]
    return (1 - lam) * l1 + lam * dss                          # [H, W]
```

Internally `_ssim_luminance`, `_ssim_contrast`, `_ssim_structure` share a single call to the window-conv backbone that produces $\mu$, $\sigma^2$, $\sigma_{xy}$ — don't run the window three times.

#### 7.5.3 Swap into the Phase 3 loss path

The Phase 3 block in `get_single_image_losses()` becomes:

```python
gt_rgb = batch["image"]                                  # [H, W, 3]

if self.config.enable_appearance_module and self.training:
    m = l_rgb_map_decoupled(
        pred_untransformed = outputs["rgb"],
        pred_adjusted      = outputs["rgb_adjusted"],
        gt                 = gt_rgb,
        lam                = self.config.lambda_rgb,
    )
elif self.config.enable_appearance_module:  # eval: module disabled, §7.5.4
    m = l_rgb_map(outputs["rgb"], gt_rgb, self.config.lambda_rgb)
else:
    m = l_rgb_map(outputs["rgb"], gt_rgb, self.config.lambda_rgb)

if (self.config.enable_confidence
        and self.step >= self.config.confidence_start_iter):
    c_hat = outputs["confidence_map"].clamp(min=1e-6)    # [H, W]
    loss_dict["rgb"] = (
        m * c_hat - self.config.confidence_beta * torch.log(c_hat)
    ).mean()
else:
    loss_dict["rgb"] = m.mean()
```

The elegance is that **confidence weighting and appearance decoupling compose cleanly**: `m` is a per-pixel map regardless of which features are on, and `Ĉ` multiplies it elementwise as before. This answers Part 5's open question #1 about multiview + confidence: yes, appearance × confidence nests naturally because both live in the per-pixel `L_rgb` map.

#### 7.5.4 Eval / inference behavior

At eval time, the embedding for a **held-out** camera does not exist — VastGaussian has no principled story for this. The paper's eval protocol (Tab. 1) follows Mip-NeRF 360's trick: optimize a small correction on the *left half* of each test image, then measure on the right half. This is:

- **Out of scope for this first integration.** Implementing it requires mid-eval optimization, which nerfstudio's `Evaluator` is not built for.
- **Reasonable fallback**: run eval with `M = 1` (identity). Reported eval PSNR/SSIM will be pessimistic for scenes with strong photometric drift but correctly measures **geometric consistency** — which is arguably what we actually want to know.

Control this via `appearance_discard_at_eval=True` (default). At eval, `get_outputs()` returns only `outputs["rgb"]`; the loss path's `self.training` check naturally falls through to the vanilla `l_rgb_map` branch.

If later we want paper-faithful eval numbers, add a new method `evaluate_with_half_image_fit()` that runs one mini-optimization per test image before measuring.

### 7.6 Phase 7D — Interaction with Phases 3, 5, 6 (confidence, variance losses)

#### 7.6.1 Confidence × appearance (critical design decision)

Both `Ĉ` and the appearance module compensate for "hard to fit" regions. They overlap but are not redundant:

- **Confidence** is *per-Gaussian*, view-invariant — it identifies primitives that are globally uncertain (specular surfaces, thin structures).
- **Appearance** is *per-image*, view-dependent — it identifies **cross-image drift** (exposure, white balance).

They should compose, not conflict. The recommended wiring (already in §7.5.3): `L_rgb` is computed with appearance decoupling → then multiplied pointwise by `Ĉ`. Specifically:

$$
\mathcal{L}_{\text{full}}(r) \;=\; \underbrace{\big[(1-\lambda)\,L_1(\mathcal{I}_i^a,\mathcal{I}_i)(r) + \lambda\,(1 - l^a c^r s^r)(r)\big]}_{\text{appearance-decoupled } L_{\text{rgb}}(r)} \cdot \hat{C}(r) \;-\; \beta\,\log \hat{C}(r)
$$

**Gotcha**: the per-Gaussian confidence gradient still flows through the photometric term. With appearance enabled, some pixels that used to look "hard" (bright-sky exposure drift) are now well-explained by $\mathcal{M}_i$, so $\hat{C}$ should converge to *higher* values over those regions. This is a desirable outcome — the ablation in §7.7 should show reduced confidence-map activity on sky / ground vs. baseline CoMe, and confidence map activity concentrated on genuinely hard primitives.

#### 7.6.2 Variance losses

`L_normal-var` and `L_color-var` (Phases 5 and 6) are **independent of the appearance module** — they act on per-Gaussian colors/normals, not on rendered images. Keep them unchanged.

One caveat for `L_color-var` (Phase 6): it compares per-Gaussian SH-evaluated color $c_i$ to the rendered $C(r)$. Both are in the **untransformed** color space — do *not* apply $\mathcal{M}$ to either, or the loss becomes meaningless (it would measure how well the CNN's 2D transform happens to factor into a per-Gaussian consensus, which is not what we want).

#### 7.6.3 Multiview photometric losses (visiofacto-specific)

Visiofacto's `get_multiview_losses()` compares renders across neighboring views. Each neighbor has its own embedding $\ell_j$ and transform $\mathcal{M}_j$. Options:

- **(Recommended) Use untransformed renders for multiview consistency**. The multiview loss is a geometry regularizer — applying per-view transforms would absorb the cross-view discrepancy it is trying to expose. Leave `get_multiview_losses()` untouched; it consumes `outputs["rgb"]`, not `outputs["rgb_adjusted"]`.
- Alternative: compare transformed renders to transformed ground-truth pairs. Strictly worse — we lose the cross-view signal we wanted.

This is the same principle as CoMe's SSIM decoupling: **geometric checks should see untransformed renders; photometric fit can see transformed ones.**

### 7.7 Phase 7E — Validation

Add to the Phase 4 (Part 4) checks:

- **Unit test: zero-init identity.** With a freshly constructed `AppearanceTransformCNN`, `forward(any_input)` must return a tensor that is **exactly zero** (not just small — the head's final conv is zero-initialized). Consequence: `exp(M) = 1` everywhere, so `rgb_adjusted == rgb_render` at iter 0 to bit-exactness. This is the strongest signal that the port has not introduced random-init drift.
- **Unit test: detach severs the gradient path.** Construct a toy loss `(rgb_adjusted.sum()).backward()` and verify that `model.gauss_params["means"].grad is None` (or zero) when `appearance_detach_cnn_input=True`. The module can still learn (its own params get gradients), but the Gaussians must not.
- **Unit test: `ssim_components(a, b)` product `l·c·s`** must equal `ssim_map(a, b)` to `1e-6` absolute tolerance on random inputs. Run this before enabling `appearance_use_ssim_decoupling`.
- **Unit test: reflection-pad round-trip.** For `H, W` not multiples of 32, verify the pad→downsample→CNN→crop pipeline preserves the original `(H, W)` output shape, and that the zero-init identity still holds post-crop.
- **At iter 500 (post warm-up)** with appearance on: log the histogram of $\mathcal{M}$ values. Expect a tight unimodal distribution around $0$ (most pixels need small corrections); long tails indicate the module has found large-scale appearance drift (expected in datasets with exposure changes) or is overfitting (bad — investigate).
- **Mid-training visualization**: log $\mathcal{M}_i$ for a handful of training images. Expect smooth low-frequency fields (per-image exposure/color-cast corrections). High-frequency structure in $\mathcal{M}$ = module is leaking geometry detail — bug.
- **Train vs eval PSNR gap**: with `appearance_discard_at_eval=True`, the gap should be larger than without the module (because training benefits from the per-image fit). This is expected; the interesting metric is **eval F1 on Tanks & Temples**, which should improve over baseline CoMe on scenes with appearance variation (ScanNet++ is the discriminator per CoMe §4).

**Expected paper-faithful outcome** (CoMe Tab. 1, ScanNet++):

- Baseline CoMe without appearance module: degraded F1 on scenes with exposure drift.
- CoMe + SSIM-decoupled appearance: F1 recovered, ~equal to CoMe on well-lit scenes.

If your implementation shows a *regression* on well-lit scenes when enabling appearance, the likely culprit is the SSIM decoupling (the $c, s$ maps running on the wrong image) — verify §7.5.2's invariant first.

### 7.8 Phase 7F — Ablation order (extends §3.1)

Update the stacking order:

1. Baseline visiofacto.
2. `+ enable_confidence` (Phases 1–4).
3. `+ enable_normal_variance_loss` (Phase 5).
4. `+ enable_color_variance_loss` (Phase 6).
5. `+ enable_appearance_module`, `appearance_use_ssim_decoupling=False` (VastGaussian-faithful baseline, Phase 7).
6. `+ appearance_use_ssim_decoupling=True` (full CoMe recipe).

Step 5 → step 6 isolates the SSIM-decoupling delta — critical for justifying the extra complexity. CoMe's paper does not run this exact ablation; we should, because it is cheap and answers "was the decoupling actually worth it on our data?"

### 7.9 Open questions specific to Part 7

1. **Held-out camera eval**: is it worth implementing the Mip-NeRF 360 half-image-fit protocol to get paper-faithful eval PSNR? My recommendation: no, unless we need to publish numbers against VastGaussian. For internal iteration, fixed `M=1` eval is cleaner and geometry-focused.
2. **Per-scene embedding persistence**: do we checkpoint the per-image embeddings? Yes — they are state tied to the training set and are needed to resume training. Add them to the `state_dict` alongside `gauss_params`. (They are automatically handled if we register the `nn.Embedding` as a submodule of the model.)
3. **Appearance module size at high resolution**: at 4K, the H/32 × W/32 × 256 internal tensor is ~60 MB per image. Still fine on single-image batches, but if we ever go multi-view-per-step we need to chunk. Flag: add a LOD switch if visiofacto ever trains on > 4K inputs.
4. **Does the module interact badly with visiofacto's two-stage training?** `enable_geometric_training` / `render_geometry_start` flip on geometric losses partway through. The appearance module should start from step 0 — it needs all of training to learn per-image transforms. Verify by logging $\mathcal{M}$ evolution across the stage transition.

### 7.10 Part 7 additions to the deliverables checklist

- [ ] `VisiofactoModelConfig` exposes all appearance fields with paper defaults (`activation="exp"`, `positional_encoding=True`, `detach_cnn_input=True`, `pad_mode="reflect"`)
- [ ] `appearance_embed` table and `appearance_cnn` are registered as submodules and survive checkpoint save/load
- [ ] CNN architecture passes the **zero-init identity test** (forward returns exactly zero; `exp(M) = 1`; `rgb_adjusted == rgb_render` at iter 0)
- [ ] CNN input is `.detach()`-ed before being fed to the network — verified by a gradient-path unit test
- [ ] Positional encoding `(u, v, r)` is the last 3 channels of the CNN input; CNN input dimensionality is 70 (not 67) by default
- [ ] Reflection padding handles arbitrary `(H, W)`; output is cropped back to `(H, W)` without calling `F.interpolate` on the final head output
- [ ] Transform uses `exp(M)`, not `sigmoid(M)`; swap is controlled by `appearance_activation`
- [ ] Training step correctly fetches per-image embedding by `image_idx`
- [ ] `outputs["rgb_adjusted"]` and `outputs["appearance_map"]` appear in eval visualizations at training time
- [ ] Phase 3 confidence clamp is `[1e-3, 5.0]` (both ends) — not just a lower floor
- [ ] Phase 4 densification divisor is `min(γ̃, 1)` clamped — confident Gaussians must not split on an easier threshold
- [ ] `ssim_components(...).product` matches `ssim_map(...)` to `1e-6` on random inputs (unit test)
- [ ] `l_rgb_map_decoupled` routes L1 and luminance to adjusted render, contrast/structure to untransformed — verified by reading the assembled loss graph in one step
- [ ] At eval, `outputs["rgb_adjusted"]` is absent and the loss falls through to scalar `L_rgb`
- [ ] Ablation step 5 → step 6 (decoupled SSIM) runs end-to-end on one scene with appearance drift (e.g., ScanNet++)
- [ ] Appearance module is disabled in `get_multiview_losses()` — multiview sees untransformed renders only
- [ ] Training wall-time overhead is within 10–15% of the no-appearance baseline (VastGaussian Tab. 5 measured +9% at 1080p); if larger, the CNN is probably running at the wrong resolution

---

## References

- [CoMe paper (Radl 2026)](../papers/radl2026_confidence-mesh-3dgs.md) · [arXiv](https://arxiv.org/abs/2603.24725) · [project page](https://r4dl.github.io/CoMe/) — §3.2 specifies the SSIM decoupling, §3.3–3.4 specify confidence + variance losses
- [VastGaussian paper (Lin 2024)](../papers/lin2024_vastgaussian.md) · [arXiv](https://arxiv.org/abs/2402.17427) — §4.2 + supplement §7 specify the decoupled appearance module CoMe builds on
- [[nerfstudio]] — codebase map this design doc is built against
- [SOF paper (Radl 2025)](../papers/radl2025_sof.md) — CoMe inherits the mesh extraction pipeline from SOF
- [[gaussian-to-mesh-pipelines]] — broader context for where CoMe sits
